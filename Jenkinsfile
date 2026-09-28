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

        stage('Compile, Audit, and Test Application') {
            steps {
                echo 'Launching stable container to execute framework metrics audits and unit tests...'
                script {
                    // Executes everything inside a single container environment to maintain the PHP 7.4 runtime path variables
                    sh '''
                        docker run --rm \
                          -v ${WORKSPACE}:/app \
                          -v ${WORKSPACE}/tmp_bin/composer:/usr/local/bin/composer \
                          -w /app \
                          -e DEBIAN_FRONTEND=noninteractive \
                          ubuntu:20.04 \
                          bash -c "apt-get update -qq && apt-get install -y -qq php-cli php-mysql php-xml php-mbstring php-zip unzip && /usr/local/bin/composer install --no-interaction --prefer-dist --ignore-platform-reqs && php artisan key:generate && php artisan migrate --force && mkdir -p build/logs && echo 'Lines of Code (LOC),Directories,Files' > build/logs/phploc.csv && echo \\$(find app -type f -name '*.php' | xargs cat | wc -l),\\$(find app -type d | wc -l),\\$(find app -type f -name '*.php' | wc -l) >> build/logs/phploc.csv && echo 'Y_AXIS_FILES='\\$(find app -name '*.php' | wc -l) > plot.properties && echo 'Y_AXIS_LINES='\\$(find app -name '*.php' | xargs cat | wc -l) >> plot.properties && echo '=== EXECUTING PHPUNIT UNIT TESTING MATRIX ===' && ./vendor/bin/phpunit && chown -R 105:109 /app"
                        
                        # Clean up temporary binary directories on host disk
                        rm -rf tmp_bin
                    '''
                }
            }
        }

        stage('Generate Trend Plots') {
            steps {
                echo 'Executing Plot Plugin Trend Analysis over generated code properties...'
                script {
                    // FIXED: Corrected parameter key names to parse data parameters into the Jenkins UI dashboard
                    plot csvFileName: 'plot-code-metrics.csv', 
                         group: 'Code Quality Metrics', 
                         title: 'Lines of Code vs Total Files Trend', 
                         style: 'line', 
                         propertiesSeries: [[file: 'plot.properties', label: 'Total Files Analyzed', key: 'Y_AXIS_FILES'], 
                                            [file: 'plot.properties', label: 'Total Lines of Code', key: 'Y_AXIS_LINES']]
                }
            }
        }
    }
}
