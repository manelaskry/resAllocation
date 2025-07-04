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
                    
                    REM Wait for container
                    :wait_loop
                    docker-compose ps backenddd | findstr "healthy" >nul
                    if errorlevel 1 (
                        echo "Waiting for container..."
                        ping 127.0.0.1 -n 6 > nul
                        goto wait_loop
                    )
                    
                    echo "=== DEBUGGING ==="
                    echo "Current directory:"
                    docker-compose exec -T backenddd pwd
                    
                    echo "Files in current directory:"
                    docker-compose exec -T backenddd ls -la
                    
                    echo "Checking vendor folder:"
                    docker-compose exec -T backenddd ls -la vendor/ || echo "No vendor folder"
                    
                    echo "Checking vendor/bin:"
                    docker-compose exec -T backenddd ls -la vendor/bin/ || echo "No vendor/bin folder"
                    
                    echo "Looking for composer.json:"
                    docker-compose exec -T backenddd ls -la composer.json || echo "No composer.json"
                    
                    echo "=== INSTALLING DEPENDENCIES ==="
                    docker-compose exec -T backenddd composer install --no-interaction
                    
                    echo "=== RUNNING TESTS ==="
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
