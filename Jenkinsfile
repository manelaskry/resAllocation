pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Test') {
            steps {
                bat '''
                    docker-compose up -d backenddd
                    
                    REM Fix timeout command for Windows
                    timeout /t 30 /nobreak
                    
                    REM Install composer first, then dependencies
                    docker-compose exec -T backenddd php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
                    docker-compose exec -T backenddd php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                    docker-compose exec -T backenddd rm composer-setup.php
                    
                    REM Now install dependencies
                    docker-compose exec -T backenddd composer install --no-interaction
                    
                    REM Run tests
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/
                    
                    REM Cleanup
                    docker-compose down
                '''
            }
        }
    }
    
    post {
        always {
            bat 'docker-compose down || exit /b 0'
        }
        success { echo 'Tests passed ✅' }
        failure { echo 'Tests failed ❌' }
    }
}
