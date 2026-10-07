node {
    // Track unique, immutable build versions via Jenkins internal build metrics
    def appVersion = "1.0.${BUILD_NUMBER}"
    
    stage('Checkout SCM') {
        // SECURE CORRECTION: Replaces 777 permissions with secure ownership mappings and a 755 mask
        sh """
            sudo chown -R jenkins:jenkins ${WORKSPACE}
            sudo chmod -R 755 ${WORKSPACE}
        """
        checkout scm
    }

    stage('Execute Genuine Unit Tests & Coverage') {
        // Generates actual test reports, eliminating the line-of-code metric placeholder issue
        sh """
            composer install --no-interaction --prefer-dist --ignore-platform-reqs
            mkdir -p build/logs bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
            chmod -R 755 bootstrap/cache storage
            ./vendor/bin/phpunit --coverage-clover build/logs/clover.xml --log-junit build/logs/junit.xml
        """
    }

    stage('SonarQube Static Code Analysis') {
        // Securely pulling credentials from the Jenkins Vault without cleartext exposure
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SECURE_TOKEN')]) {
            def scannerHome = tool 'SonarQubeScanner'
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=php-todo \
                -Dsonar.projectName=php-todo \
                -Dsonar.host.url=http://50.19.60.165:9000 \
                -Dsonar.login=${SECURE_TOKEN} \
                -Dsonar.sources=. \
                -Dsonar.exclusions=**/vendor/**,**/tests/** \
                -Dsonar.php.coverage.reportPaths=build/logs/clover.xml \
                -Dsonar.php.tests.reportPath=build/logs/junit.xml"
            }
        }
    }

    stage('Package Neutral Artifact') {
        // SECURE CORRECTION: Explicitly excludes all configuration state assets and secrets (.env) from packaging
        sh "tar --exclude='.git' --exclude='.env' --exclude='tests' -czf php-todo-${appVersion}.tar.gz ."
    }

    stage('Publish Versioned Build to JFrog Locker') {
        // Secure authentication via Port 8082 API pathways using your vault credential mapping
        withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CREDS', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
            echo "Uploading immutable build artifact [${appVersion}] to centralized repository storage..."
            sh "curl -u ${JF_USER}:${JF_PASS} -T php-todo-${appVersion}.tar.gz 'http://34.227.205.86:8082/artifactory/generic-local-repo/php-todo-${appVersion}.tar.gz'"
        }
    }

    stage('Trigger Downstream Infrastructure Deployment') {
        // propagate: true guarantees that any downstream deployment issues will visibly fail this parent pipeline
        build job: 'ansible-webserver-deployment', 
              parameters: [string(name: 'ARTIFACT_VERSION', value: appVersion)], 
              wait: true, 
              propagate: true
    }
}
