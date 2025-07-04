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
                timeout(time: 15, unit: 'MINUTES') {
                    bat '''
                        REM Clean up
                        docker-compose down --volumes --remove-orphans || exit /b 0
                        
                        REM Start containers
                        docker-compose up -d
                        
                        REM Wait for containers
                        echo "Waiting for containers to start..."
                        ping 127.0.0.1 -n 121 > nul
                        
                        REM Check container status
                        docker-compose ps
                        
                        REM Check if backend container exists
                        docker-compose ps backenddd || (
                            echo "Backend container not found, checking logs:"
                            docker-compose logs backenddd
                            exit /b 1
                        )
                        
                        REM Run tests
                        docker-compose exec -T backenddd ./vendor/bin/phpunit tests/ --testdox
                        
                        REM Cleanup
                        docker-compose down
                    '''
                }
            }
        }
    }
    
    post {
        always {
            bat '''
                echo "=== Container Status ==="
                docker-compose ps || exit /b 0
                echo "=== Backend Logs ==="
                docker-compose logs backenddd || exit /b 0
                docker-compose down --volumes --remove-orphans || exit /b 0
            '''
        }
        success { echo 'Tests passed ✅' }
        failure { echo 'Tests failed ❌' }
    }
}