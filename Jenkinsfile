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

        stage('Prepare and Test Application inside Container') {
            steps {
                echo 'Renaming configuration profile...'
                sh 'mv .env.sample .env'
                
                echo 'Launching isolated PHP 7.0 workspace container to execute test matrices...'
                // Spins up a clean, isolated PHP 7 environment to insulate scripts from host server components
                sh '''
                    docker run --rm \
                      -v ${WORKSPACE}:/app \
                      -w /app \
                      -e PDO_MYSQL_ATTR_SSL_CA=false \
                      textik/php70-composer:latest \
                      bash -c "composer install --no-interaction --prefer-dist && ./vendor/bin/phpunit"
                '''
            }
        }
    }
}
