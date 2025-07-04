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
                        
                        REM MANUALLY install dependencies
                        echo "Installing composer dependencies..."
                        docker-compose exec -T backenddd composer install --no-interaction --optimize-autoloader
                        
                        REM Verify PHPUnit exists
                        docker-compose exec -T backenddd ls -la vendor/bin/phpunit
                        
                        REM Run tests
                        echo "Running tests..."
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
            bat 'docker-compose down --volumes --remove-orphans || exit /b 0'
        }
        success { echo 'Tests passed ✅' }
        failure { 
            echo 'Tests failed ❌'
            bat '''
                echo "=== Backend Logs ==="
                docker-compose logs backenddd || exit /b 0
            '''
        }
    }
}
