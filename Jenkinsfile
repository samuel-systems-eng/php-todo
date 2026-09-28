pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/samuel-systems-eng/php-todo.git'
            }
        }

        stage('Prepare Dependencies') {
            steps {
                echo 'Renaming configuration profiles and bootstrapping environment folders...'
                sh 'mv .env.sample .env'
                
                script {
                    sh '''
                        # 1. Setup local composer binary bridges
                        mkdir -p tmp_bin
                        docker run --rm --entrypoint cat composer:1 /usr/bin/composer > tmp_bin/composer
                        chmod +x tmp_bin/composer
                        
                        # 2. Build the structural Laravel cache directory trees
                        mkdir -p bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing
                        chmod -R 777 bootstrap/cache storage
                    '''
                }
            }
        }

        stage('Compile and Audit Codebase') {
            steps {
                echo 'Launching container to execute framework installation and PHP Insights audits...'
                script {
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -v ${WORKSPACE}/tmp_bin/composer:/usr/local/bin/composer \
                          -w /app \
                          -e DEBIAN_FRONTEND=noninteractive \
                          ubuntu:20.04 \
                          bash -c "apt-get update -qq && apt-get install -y -qq php-cli php-mysql php-xml php-mbstring php-zip unzip && /usr/local/bin/composer install --no-interaction --prefer-dist --ignore-platform-reqs && /usr/local/bin/composer require nunomaduro/phpinsights:v1.14.0 --dev --no-interaction --ignore-platform-reqs && php artisan key:generate && php artisan migrate --force && echo '=== RUNNING PHP INSIGHTS CODE METRICS ===' && ./vendor/bin/phpinsights analyse --no-interaction && chown -R 105:109 /app"
                    '''
                }
            }
        }

        stage('Execute Unit Tests') {
            steps {
                echo 'Invoking dedicated PHPUnit testing matrices...'
                script {
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -w /app \
                          ubuntu:20.04 \
                          bash -c "./vendor/bin/phpunit"
                    '''
                }
            }
        }
    }
}
