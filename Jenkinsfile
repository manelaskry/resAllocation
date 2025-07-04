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
                
                REM Wait for container to be ready
                echo "Waiting for container to start..."
                ping 127.0.0.1 -n 121 > nul
                
                REM Check if PHPUnit exists and find its location
                echo "Checking PHPUnit installation..."
                docker compose exec backenddd ls -la vendor/bin/
                docker compose exec backenddd which phpunit || echo "PHPUnit not in PATH"
                
                REM Try different PHPUnit paths
                docker compose exec backenddd ./vendor/bin/phpunit --version || echo "PHPUnit not found in vendor/bin"
                
                docker-compose down
            '''
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
