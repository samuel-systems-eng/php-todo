node {
    def appVersion = "1.0.${BUILD_NUMBER}"
    
    stage('Checkout SCM') {
        checkout scm
    }

    stage('Execute Genuine Unit Tests & Coverage') {
        // PERMANENT FIX: We removed the host-level chmod that causes permission blocks.
        // Instead, the container runs 'chmod -R 777' from the INSIDE as root over the shared storage volumes,
        // safely cleaning up and resetting permissions without crashing Jenkins.
        sh """
            mkdir -p build/logs bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
            
            docker run --rm -v \${WORKSPACE}:/app -w /app php:7.4-cli-alpine sh -c "chmod -R 777 bootstrap/cache storage && rm -rf storage/framework/sessions/* && apk add --no-cache bash mariadb-dev bzip2-dev autoconf g++ make && docker-php-ext-install pdo_mysql && pecl install pcov && docker-php-ext-enable pcov && ./vendor/bin/phpunit --coverage-clover build/logs/clover.xml --log-junit build/logs/junit.xml"
        """
    }

    stage('SonarQube Static Code Analysis') {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SECURE_TOKEN')]) {
            def scannerHome = tool 'SonarQubeScanner'
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=php-todo -Dsonar.projectName=php-todo -Dsonar.host.url=http://50.19.60.165:9000 -Dsonar.login=\${SECURE_TOKEN} -Dsonar.sources=. -Dsonar.exclusions=**/vendor/**,**/tests/** -Dsonar.php.coverage.reportPaths=build/logs/clover.xml -Dsonar.php.tests.reportPath=build/logs/junit.xml"
            }
        }
    }

    stage('Package Neutral Artifact') {
        sh "tar --exclude='.git' --exclude='.env' --exclude='tests' -czf ../php-todo-\${appVersion}.tar.gz ."
    }

    stage('Publish Versioned Build to JFrog Locker') {
        withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CREDS', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
            echo "Uploading immutable build artifact [\${appVersion}] to centralized repository storage..."
            sh "curl -u \${JF_USER}:\${JF_PASS} -T ../php-todo-\${appVersion}.tar.gz 'http://34.227.205.86:8082/artifactory/generic-local-repo/php-todo-${appVersion}.tar.gz'"
            sh "rm -f ../php-todo-\${appVersion}.tar.gz"
        }
    }

    stage('Trigger Downstream Infrastructure Deployment') {
        build job: 'Multibranch pipeline/develop', 
              parameters: [string(name: 'ARTIFACT_VERSION', value: appVersion)], 
              wait: true, 
              propagate: true
    }
}