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
                
                echo 'Launching certified public Composer 1.x container layer...'
                script {
                    // Invokes the official stable Composer v1 image architecture
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e PDO_MYSQL_ATTR_SSL_CA=false \
                          composer:1.10 \
                          bash -c "composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit && chown -R $(id -u):\$(id -g) /app"
                    '''
                }
            }
        }
    }
}