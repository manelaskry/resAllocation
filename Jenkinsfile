pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')  // Run daily at 2 AM
    }

    stages {
        stage('Test') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
                
                bat '''
                    REM Clean up any existing containers
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker container prune -f || exit /b 0
                    
                    REM Start the backend container
                    docker-compose up -d backenddd
                    
                    REM Wait 2 minutes for container to fully initialize
                    echo "Waiting for container setup to complete..."
                    ping 127.0.0.1 -n 121 > nul
                    
                    REM Run PHPUnit tests
                    echo "Running PHPUnit tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    
                    REM Clean up
                    docker-compose down
                '''
            }
        }
    }
    
    post {
        always {
            bat '''
                docker-compose down --volumes --remove-orphans || exit /b 0
            '''
        }
        success {
            echo 'PHPUnit tests passed successfully! ✅'
        }
        failure {
            echo 'PHPUnit tests failed ❌'
        }
    }
}
