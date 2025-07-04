pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')
    }

    stages {
        stage('Test') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
                
                bat '''
                    docker-compose up -d backenddd
                    ping 127.0.0.1 -n 61 > nul
                    
                    REM Install composer in container
                    docker-compose exec -T backenddd curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
                    
                    REM Install dependencies
                    docker-compose exec -T backenddd composer install --no-interaction
                    
                    REM Run tests
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    
                    docker-compose down
                '''
            }
        }
    }
    
    post {
        always {
            bat 'docker-compose down || exit /b 0'
        }
        success { echo '34 Entity tests passed! ✅' }
        failure { echo 'Tests failed ❌' }
    }
}
