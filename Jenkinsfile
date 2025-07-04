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
                    REM Install dependencies using composer image
                    docker run --rm -v "%cd%\\backenddd":/app -w /app composer:latest install --no-interaction
                    
                    REM Start container and run tests
                    docker-compose up -d backenddd
                    ping 127.0.0.1 -n 61 > nul
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    docker-compose down
                '''
            }
        }
    }
}
