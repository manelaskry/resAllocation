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
                    
                    REM Wait for container to be ready (2 minutes)
                    echo "Waiting for container to start..."
                    ping 127.0.0.1 -n 121 > nul
                    
                    REM Run tests directly (dependencies already installed)
                    echo "Running PHPUnit tests..."
                    docker compose exec backenddd ./vendor/bin/phpunit tests/Entity/ProjectTest.php –testdox
                    
                    docker-compose down
                '''
            }
        }
    }
    
    post {
        always {
            bat 'docker-compose down || exit /b 0'
        }
        success { 
            echo 'All 34 tests passed! ✅'
        }
        failure { 
            echo 'Tests failed ❌'
            bat 'docker-compose logs backenddd || exit /b 0'
        }
    }
}
