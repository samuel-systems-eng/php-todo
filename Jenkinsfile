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
                // Tracking your clean personal application repo branch target
                git branch: 'main', url: 'https://github.com/samuel-systems-eng/php-todo.git'
            }
        }
        stage('Prepare Dependencies') {
            steps {
                sh 'mv .env.sample .env'
                sh 'composer install'
                // ENFORCED GLOBAL BYPASS OVERRIDE ADDED INLINE BELOW:
                sh 'PDO_MYSQL_ATTR_SSL_CA=false php artisan migrate --force'
                sh 'php artisan db:seed --force'
                sh 'php artisan key:generate'
            }
        }

        stage('Execute Unit Tests') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }
    }
}
