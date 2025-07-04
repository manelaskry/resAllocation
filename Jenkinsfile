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
                    REM Clean up first
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker container prune -f || exit /b 0
                    
                    REM Start containers
                    docker-compose up -d backenddd
                    
                    REM Wait with timeout (max 5 minutes = 30 attempts)
                    set /a attempts=0
                    :wait_setup
                    set /a attempts+=1
                    if %attempts% gtr 30 (
                        echo "Timeout! Setup took too long. Debugging..."
                        docker-compose logs backenddd
                        docker-compose exec -T backenddd ls -la /tmp/
                        docker-compose exec -T backenddd ps aux
                        goto cleanup
                    )
                    
                    docker-compose exec -T backenddd test -f /tmp/setup_done >nul 2>&1 && goto setup_complete
                    echo "Waiting for setup... (attempt %attempts%/30)"
                    ping 127.0.0.1 -n 11 > nul
                    goto wait_setup
                    
                    :setup_complete
                    echo "Setup completed! Running tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    goto cleanup
                    
                    :cleanup
                    docker-compose down
                '''
            }
        }
    }
}
