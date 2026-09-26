pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/samuel-systems-eng/php-todo.git'
            }
        }

        stage('Prepare and Test Application') {
            steps {
                echo 'Renaming environment configuration file...'
                sh 'mv .env.sample .env'
                
                echo 'Launching stable container with isolated database bootstrapping...'
                script {
                    sh '''
                        # 1. Create a local temporary directory on the host server
                        mkdir -p tmp_bin
                        
                        # 2. Extract the working, native composer binary directly out of the official image
                        docker run --rm --entrypoint cat composer:1 /usr/bin/composer > tmp_bin/composer
                        chmod +x tmp_bin/composer
                        
                        # 3. Mount code and binary safely inside the root-level container to execute setups
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -v ${WORKSPACE}/tmp_bin/composer:/usr/local/bin/composer \
                          -w /app \
                          -e DEBIAN_FRONTEND=noninteractive \
                          ubuntu:20.04 \
                          bash -c "apt-get update -qq && apt-get install -y -qq php-cli php-mysql php-xml php-mbstring php-zip unzip && mkdir -p bootstrap/cache storage/framework/sessions storage/framework/views storage/framework/testing && chmod -R 777 bootstrap/cache storage && composer install --no-interaction --prefer-dist --ignore-platform-reqs && php artisan key:generate && php artisan migrate --force && ./vendor/bin/phpunit && chown -R 105:109 /app"
                        
                        # 4. Clean up our temporary binary directory post-execution
                        rm -rf tmp_bin
                    '''
                }
            }
        }
    }
}
