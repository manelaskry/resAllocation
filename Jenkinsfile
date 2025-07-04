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
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker-compose up -d backenddd
                    
                    REM Wait for the container's setup script to complete
                    echo "Waiting for container setup to complete..."
                    set /a attempts=0
                    :wait_for_setup
                    set /a attempts+=1
                    if %attempts% GEQ 60 (
                        echo "Setup timeout after 10 minutes!"
                        docker-compose logs backenddd
                        goto cleanup
                    )
                    
                    docker-compose exec -T backenddd test -f /tmp/setup_done >nul 2>&1 && goto setup_ready
                    echo "Setup in progress... (attempt %attempts%/60)"
                    ping 127.0.0.1 -n 11 > nul
                    goto wait_for_setup
                    
                    :setup_ready
                    echo "Setup completed! Verifying installation..."
                    docker-compose exec -T backenddd which composer
                    docker-compose exec -T backenddd ls -la vendor/bin/phpunit
                    
                    echo "Running PHPUnit tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    
                    :cleanup
                    docker-compose down
                '''
            }
        }
    }
}
