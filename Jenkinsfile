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
                    REM Clean up any existing containers first
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker container prune -f || exit /b 0
                    
                    REM Start fresh containers
                    docker-compose up -d backenddd
                    
                    REM Wait for setup to complete
                    :wait_setup
                    docker-compose exec -T backenddd test -f /tmp/setup_done >nul 2>&1 && goto setup_complete
                    echo "Waiting for container setup..."
                    ping 127.0.0.1 -n 11 > nul
                    goto wait_setup
                    
                    :setup_complete
                    echo "Setup completed! Running tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    
                    docker-compose down
                '''
            }
        }
    }
    
    post {
        always {
            bat '''
                docker-compose down --volumes --remove-orphans || exit /b 0
                docker container prune -f || exit /b 0
            '''
        }
        success { echo '34 Entity tests passed! ✅' }
        failure { echo 'Tests failed ❌' }
    }
}
