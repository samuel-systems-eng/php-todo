pipeline {
    agent any

    stages {
        stage("Initial cleanup") {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }
  
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com'
            }
        }

        stage('Prepare and Test Application') {
            steps {
                echo 'Renaming environment configuration file...'
                sh 'mv .env.sample .env'
                
                echo 'Launching verified PHP 7.0 + Composer container layer...'
                script {
                    // Invokes the verified public PHP 7 build container tracking layout
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e PDO_MYSQL_ATTR_SSL_CA=false \
                          edvordo/php70-composer:latest \
                          bash -c "composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}


