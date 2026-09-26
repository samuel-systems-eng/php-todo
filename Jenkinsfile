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
                
                echo 'Launching stable Ubuntu 20.04 container as current user...'
                script {
                    // Running with explicit host user IDs to prevent root-owned folder lockouts completely
                    sh '''
                        docker run --rm \
                          --user $(id -u):\$(id -g) \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          -e DEBIAN_FRONTEND=noninteractive \
                          ubuntu:20.04 \
                          bash -c "apt-get update -qq && apt-get install -y -qq php-cli php-mysql php-xml php-mbstring php-zip unzip curl && curl -sS https://getcomposer.org -o /tmp/composer && chmod +x /tmp/composer && /tmp/composer install --no-interaction --prefer-dist --ignore-platform-reqs && ./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}