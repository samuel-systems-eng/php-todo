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
                git branch: 'main', url: 'https://github.com/samuel-systems-eng/php-todo.git'
            }
        }

        stage('Prepare and Test Application') {
            steps {
                echo 'Renaming environment configuration file...'
                sh 'mv .env.sample .env'
                
                echo 'Launching dedicated PHP 7.3 container to run dependencies and testing suites...'
                script {
                    // Uses an official, highly optimized, non-deprecated PHP 7.3+Composer build image
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e PDO_MYSQL_ATTR_SSL_CA=false \
                          composer:1.10-php73 \
                          bash -c "composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}