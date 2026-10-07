node {
    def appVersion = "1.0.${BUILD_NUMBER}"
    
    stage('Checkout SCM') {
        // Enforces standard permissions using native master node user controls
        sh "chown -R jenkins:jenkins \${WORKSPACE} && chmod -R 755 \${WORKSPACE}"
        checkout scm
    }

    stage('Execute Genuine Unit Tests & Coverage') {
        // HARDENED IMPLEMENTATION: 
        // 1. Installs persistent extensions installer utility (install-php-extensions)
        // 2. Injects native pdo_mysql database drivers to satisfy Laravel connection abstractions
        // 3. Spins up a lightning-fast pcov coverage engine to generate real clover.xml matrices
        sh """
            mkdir -p build/logs bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
            chmod -R 775 bootstrap/cache storage
            
            docker run --rm -v \${WORKSPACE}:/app -w /app php:7.4-cli-alpine sh -c "apk add --no-cache bash curl && wget -q https://github.com -O /usr/local/bin/install-php-extensions && chmod +x /usr/local/bin/install-php-extensions && install-php-extensions pdo_mysql pcov && ./vendor/bin/phpunit --coverage-clover build/logs/clover.xml --log-junit build/logs/junit.xml"
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
        // SECURE CORRECTION: Explicitly excludes all cleartext secrets (.env) to protect credentials
        sh "tar --exclude='.git' --exclude='.env' --exclude='tests' -czf php-todo-\${appVersion}.tar.gz ."
    }

    stage('Publish Versioned Build to JFrog Locker') {
        withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CREDS', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
            echo "Uploading immutable build artifact [\${appVersion}] to centralized repository storage..."
            sh "curl -u \${JF_USER}:\${JF_PASS} -T php-todo-\${appVersion}.tar.gz 'http://34.227.205.86:8082/artifactory/generic-local-repo/php-todo-${appVersion}.tar.gz'"
        }
    }

    stage('Trigger Downstream Infrastructure Deployment') {
        // propagate: true guarantees full pipeline transparency upon deployment failures
        build job: 'ansible-webserver-deployment', 
              parameters: [string(name: 'ARTIFACT_VERSION', value: appVersion)], 
              wait: true, 
              propagate: true
    }
}