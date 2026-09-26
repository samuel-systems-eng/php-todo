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
                
                echo 'Executing lightweight native PHP installation and test suites...'
                // Suppresses deprecation exceptions to allow seamless execution on the host engine
                sh 'php -d error_reporting="E_ALL & ~E_DEPRECATED & ~E_NOTICE" /usr/bin/composer install --no-interaction --prefer-dist --ignore-platform-reqs'
                sh 'php -d error_reporting="E_ALL & ~E_DEPRECATED & ~E_NOTICE" vendor/bin/phpunit'
            }
        }
    }
}
