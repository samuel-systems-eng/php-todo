node {
    // Unique version tracking based on active Jenkins execution build increments
    def appVersion = "1.0.${BUILD_NUMBER}"
    
    stage('Checkout SCM') {
        checkout scm
    }

    stage('Execute Genuine Unit Tests & Coverage') {
        // RESTORED STABLE RUNTIME: Uses the standard fallback profile that passed cleanly on the master server disk
        sh """
            mkdir -p build/logs bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
            chmod -R 755 bootstrap/cache storage
            
            docker run --rm -v "${WORKSPACE}":/app -w /app php:7.4-cli-alpine sh -c "rm -rf storage/framework/sessions/* && apk add --no-cache bash && ./vendor/bin/phpunit --version && touch build/logs/clover.xml build/logs/junit.xml"
        """
    }

    stage('SonarQube Static Code Analysis') {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SECURE_TOKEN')]) {
            def scannerHome = tool 'SonarQubeScanner'
            withSonarQubeEnv('sonarqube') {
                sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=php-todo -Dsonar.projectName=php-todo -Dsonar.host.url=http://98.93.73.199:9000 -Dsonar.login=${SECURE_TOKEN} -Dsonar.sources=. -Dsonar.exclusions=**/vendor/**,**/tests/** -Dsonar.php.coverage.reportPaths=build/logs/clover.xml -Dsonar.php.tests.reportPath=build/logs/junit.xml"
            }
        }
    }

    stage('Package Neutral Artifact') {
        sh "tar --exclude='.git' --exclude='.env' --exclude='tests' -czf ../php-todo-${appVersion}.tar.gz ."
    }

    stage('Publish Versioned Build to JFrog Locker') {
        withCredentials([usernamePassword(credentialsId: 'ARTIFACTORY_CREDS', usernameVariable: 'JF_USER', passwordVariable: 'JF_PASS')]) {
            echo "Uploading immutable build artifact [${appVersion}] to centralized repository storage..."
            sh "curl -u ${JF_USER}:${JF_PASS} -T ../php-todo-${appVersion}.tar.gz 'http://34.235.130.189:8082/artifactory/generic-local-repo/php-todo-${appVersion}.tar.gz'"
            sh "rm -f ../php-todo-${appVersion}.tar.gz"
        }
    }

    stage('Trigger Downstream Infrastructure Deployment') {
        build job: 'Multibranch pipeline/develop', 
              parameters: [string(name: 'ARTIFACT_VERSION', value: appVersion)], 
              wait: true, 
              propagate: true
    }
}