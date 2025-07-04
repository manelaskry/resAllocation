pipeline {
    agent any
    
    environment {
        COMPOSE_PROJECT_NAME = "myapp_${BUILD_NUMBER}"
        DOCKER_BUILDKIT = "1"
        COMPOSE_DOCKER_CLI_BUILD = "1"
    }
    
    triggers {
        // Run every day at 2 AM
        cron('0 2 * * *')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Environment Setup') {
            steps {
                script {
                    echo 'Setting up environment...'
                    // Clean up any existing containers
                    bat '''
                        docker-compose -p %COMPOSE_PROJECT_NAME% down --volumes --remove-orphans 2>nul || echo "No containers to clean"
                        docker system prune -f 2>nul || echo "Cleanup completed"
                    '''
                }
            }
        }
        
        stage('Build and Start Services') {
            steps {
                echo 'Building and starting services...'
                bat '''
                    REM Start all services and wait using docker-compose built-in waiting
                    docker-compose -p %COMPOSE_PROJECT_NAME% up -d
                    
                    REM Give containers time to start
                    echo "Waiting for services to start..."
                    ping 127.0.0.1 -n 31 > nul
                    
                    REM Check if containers are running
                    docker-compose -p %COMPOSE_PROJECT_NAME% ps
                '''
            }
        }
        
        stage('Wait for Database') {
            steps {
                echo 'Waiting for databases to be ready...'
                script {
                    // Retry mechanism for database readiness
                    def maxAttempts = 30
                    def attempt = 0
                    def dbReady = false
                    
                    while (!dbReady && attempt < maxAttempts) {
                        try {
                            bat '''
                                docker-compose -p %COMPOSE_PROJECT_NAME% exec -T postgres pg_isready -U root -d app
                                docker-compose -p %COMPOSE_PROJECT_NAME% exec -T postgres_test pg_isready -U root -d app_test
                            '''
                            dbReady = true
                            echo "Databases are ready!"
                        } catch (Exception e) {
                            attempt++
                            echo "Database not ready yet, attempt ${attempt}/${maxAttempts}"
                            sleep(5)
                        }
                    }
                    
                    if (!dbReady) {
                        error "Databases failed to start after ${maxAttempts} attempts"
                    }
                }
            }
        }
        
        stage('Wait for Backend') {
            steps {
                echo 'Waiting for backend to be ready...'
                script {
                    def maxAttempts = 60
                    def attempt = 0
                    def backendReady = false
                    
                    while (!backendReady && attempt < maxAttempts) {
                        try {
                            bat '''
                                docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd test -f /tmp/setup_done
                            '''
                            backendReady = true
                            echo "Backend is ready!"
                        } catch (Exception e) {
                            attempt++
                            echo "Backend not ready yet, attempt ${attempt}/${maxAttempts}"
                            sleep(3)
                        }
                    }
                    
                    if (!backendReady) {
                        bat '''
                            echo "Backend setup failed, showing logs:"
                            docker-compose -p %COMPOSE_PROJECT_NAME% logs backenddd
                        '''
                        error "Backend failed to start after ${maxAttempts} attempts"
                    }
                }
            }
        }
        
        stage('Prepare Test Environment') {
            steps {
                echo 'Preparing test environment...'
                bat '''
                    REM Fix missing dependencies first
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd composer require symfony/runtime --no-interaction || echo "Runtime package handled"
                    
                    REM Install all dependencies
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd composer install --no-interaction --optimize-autoloader
                    
                    REM Verify console works
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console --version || (
                        echo "Console not working, checking dependencies..."
                        docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd composer show | grep symfony/runtime || echo "Runtime package missing"
                        exit /b 1
                    )
                    
                    REM Create test database (ignore if exists)
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:database:create --env=test --if-not-exists || echo "Test database creation handled"
                    
                    REM Run migrations
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:migrations:migrate --env=test --no-interaction || echo "Migrations completed"
                    
                    REM Load fixtures (optional)
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:fixtures:load --env=test --no-interaction || echo "Fixtures loaded or not available"
                '''
            }
        }
        
        stage('Run PHPUnit Tests') {
            steps {
                echo 'Running PHPUnit tests...'
                bat '''
                    REM Verify PHPUnit is available
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ./vendor/bin/phpunit --version || (
                        echo "PHPUnit not found, checking installation..."
                        docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ls -la vendor/bin/
                        exit /b 1
                    )
                    
                    REM Run the tests
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ./vendor/bin/phpunit --testdox --stop-on-failure
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            bat '''
                REM Show logs for debugging
                echo "=== Backend Logs ==="
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=50 backenddd || echo "No backend logs"
                
                echo "=== Postgres Logs ==="
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=20 postgres || echo "No postgres logs"
                
                echo "=== Test Postgres Logs ==="
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=20 postgres_test || echo "No test postgres logs"
                
                REM Clean up
                docker-compose -p %COMPOSE_PROJECT_NAME% down --volumes --remove-orphans || echo "Cleanup completed"
            '''
        }
        
        success {
            echo 'All tests passed! ✅'
        }
        
        failure {
            echo 'Tests failed ❌'
            bat '''
                echo "=== Container Status ==="
                docker-compose -p %COMPOSE_PROJECT_NAME% ps || echo "No containers running"
                
                echo "=== Detailed Backend Logs ==="
                docker-compose -p %COMPOSE_PROJECT_NAME% logs backenddd || echo "No detailed logs available"
            '''
        }
    }
}