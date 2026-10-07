node {
    def appVersion = "1.0.${BUILD_NUMBER}"
    
    stage('Checkout SCM') {
        // FIXED USER EXCEPTION: Using ubuntu:ubuntu to match the slave node's native architecture accounts
        sh "sudo chown -R ubuntu:ubuntu \${WORKSPACE} && sudo chmod -R 755 \${WORKSPACE}"
        checkout scm
    }

    stage('Execute Genuine Unit Tests & Coverage') {
        // FIXED LEGACY ENVIRONMENT CONSTRAINT: Launching an isolated container wrapper to execute testing frameworks safely
        sh """
            mkdir -p build/logs bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
            sudo chmod -R 775 bootstrap/cache storage
            
            docker run --rm \
              -v \${WORKSPACE}:/app \
              -w /app \
              php:7.4-cli-alpine \
              sh -c "apk add --no-cache bash && corepack enable && ./vendor/bin/phpunit --coverage-clover build/logs/clover.xml --log-junit build/logs/junit.xml"
        """
    }

    stage('SonarQube Static Code Analysis') {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SECURE_TOKEN')]) {
            def scannerHome = tool 'SonarQubeScanner'
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=php-todo \
                -Dsonar.projectName=php-todo \
                -Dsonar.host.url=http://50.19.60.165:9000 \
                -Dsonar.login=\${SECURE_TOKEN} \
                -Dsonar.sources=. \
                -Dsonar.exclusions=**/vendor/**,**/tests/** \
                -Dsonar.php.coverage.reportPaths=build/logs/clover.xml \
                -Dsonar.php.tests.reportPath=build/logs/junit.xml"
            }
        }
    }

    stage('Package Neutral Artifact') {
        sh "tar --exclude='.git' --exclude='.env' --exclude='tests' -czf php-todo-\${appVersion}.tar.gz ."
    }

    stage('Publish Versioned Build to JFrog Locker') {
        withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CREDS', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
            echo "Uploading immutable build artifact [\${appVersion}] to centralized repository storage..."
            sh "curl -u \${JF_USER}:\${JF_PASS} -T php-todo-\${appVersion}.tar.gz 'http://34.227.205.86:8082/artifactory/generic-local-repo/php-todo-${appVersion}.tar.gz'"
        }
    }

    stage('Trigger Downstream Infrastructure Deployment') {
        build job: 'ansible-webserver-deployment', 
              parameters: [string(name: 'ARTIFACT_VERSION', value: appVersion)], 
              wait: true, 
              propagate: true
    }
}
