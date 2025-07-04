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
                    docker-compose up -d backend
                    
                    REM Fix timeout command for Windows
                    timeout /t 30 /nobreak
                    
                    REM Install composer first, then dependencies
                    docker-compose exec -T backend php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
                    docker-compose exec -T backend php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                    docker-compose exec -T backend rm composer-setup.php
                    
                    REM Now install dependencies
                    docker-compose exec -T backend composer install --no-interaction
                    
                    REM Run tests
                    docker-compose exec -T backend ./vendor/bin/phpunit tests/
                    
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
