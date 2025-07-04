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
                    REM Start containers first
                    docker-compose up -d backenddd
                    ping 127.0.0.1 -n 61 > nul
                    
                    REM Install composer INSIDE the running container
                    docker-compose exec -T backenddd curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
                    
                    REM Install dependencies INSIDE the container
                    docker-compose exec -T backenddd composer update --no-interaction
                    
                    REM Now run tests
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
