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
                    
                    REM Wait for healthy status
                    :wait_loop
                    docker-compose ps backenddd | findstr "healthy" >nul
                    if errorlevel 1 (
                        echo "Waiting for container to be healthy..."
                        ping 127.0.0.1 -n 6 > nul
                        goto wait_loop
                    )
                    
                    echo "Container is healthy, running tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/
                    
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
