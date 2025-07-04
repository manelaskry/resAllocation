pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Build and Test') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    bat '''
                        REM Clean up
                        docker-compose down || exit /b 0
                        
                        REM Build and start
                        docker-compose up -d --build
                        
                        REM Wait longer for containers to be ready
                        echo "Waiting 60 seconds for all services to start..."
                        timeout /t 60 /nobreak
                        
                        REM Check container status
                        docker-compose ps
                        
                        REM Try to run tests with error handling
                        docker-compose exec -T backend bash -c "
                            echo 'Container is ready, checking files...'
                            ls -la /var/www/
                            
                            if [ -d /var/www/backend ]; then
                                echo 'Using /var/www/backend directory'
                                cd /var/www/backend
                            else
                                echo 'Using /var/www directory'
                                cd /var/www
                            fi
                            
                            echo 'Installing dependencies...'
                            composer install --no-interaction --prefer-dist
                            
                            echo 'Running tests...'
                            ./vendor/bin/phpunit tests/ --testdox
                        " || (
                            echo "Command failed, showing logs:"
                            docker-compose logs backend
                            exit 1
                        )
                    '''
                }
            }
        }
    }
    
    post {
        always {
            bat 'docker-compose down || exit /b 0'
        }
    }
}
