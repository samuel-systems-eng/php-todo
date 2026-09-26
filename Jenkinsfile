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
                
                echo 'Launching stable Ubuntu 20.04 container containing native PHP 7.4 runtime environment...'
                script {
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e DEBIAN_FRONTEND=noninteractive \
                          ubuntu:20.04 \
                          bash -c "apt-get update -qq && apt-get install -y -qq php-cli php-mysql php-xml php-mbzip php-zip unzip curl && curl -sS https://getcomposer.org -o /usr/local/bin/composer && chmod +x /usr/local/bin/composer && composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit && chown -R \$(id -u):\$(id -g) /app"
                    '''
                }
            }
        }
    }
}