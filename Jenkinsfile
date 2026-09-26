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
                
                echo 'Launching certified public PHP 7 image workspace matrix...'
                script {
                    // Invokes the official, open-access PHP 7 engine layer
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e PDO_MYSQL_ATTR_SSL_CA=false \
                          php:7.0-cli \
                          bash -c "apt-get update && apt-get install -y unzip && curl -sS https://getcomposer.org | php -- --install-dir=/usr/local/bin --filename=composer && composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}
