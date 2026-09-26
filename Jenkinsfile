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
                
                echo 'Launching official PHP 7.3 container layer...'
                script {
                    // Pulls the guaranteed, open-access public core PHP 7.3 image
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e PDO_MYSQL_ATTR_SSL_CA=false \
                          php:7.3-cli \
                          bash -c "apt-get update -qq && apt-get install -y -qq unzip curl && curl -sS https://getcomposer.org | php -- --install-dir=/usr/local/bin --filename=composer && composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}
