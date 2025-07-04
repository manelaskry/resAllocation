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
                        docker-compose -p %COMPOSE_PROJECT_NAME% down --volumes --remove-orphans || echo "No containers to clean"
                        docker system prune -f || echo "Cleanup completed"
                    '''
                }
            }
        }
        
        stage('Build and Start Services') {
            steps {
                echo 'Building and starting services...'
                bat '''
                    REM Build and start only the necessary services for testing
                    docker-compose -p %COMPOSE_PROJECT_NAME% up -d postgres postgres_test
                    
                    REM Wait for databases to be ready
                    echo "Waiting for databases to be ready..."
                    timeout 120 docker-compose -p %COMPOSE_PROJECT_NAME% exec -T postgres pg_isready -U root -d app || (echo "Postgres main not ready" && exit 1)
                    timeout 120 docker-compose -p %COMPOSE_PROJECT_NAME% exec -T postgres_test pg_isready -U root -d app_test || (echo "Postgres test not ready" && exit 1)
                    
                    REM Start backend service
                    docker-compose -p %COMPOSE_PROJECT_NAME% up -d backenddd
                    
                    REM Wait for backend to be ready
                    echo "Waiting for backend to be ready..."
                    timeout 180 docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd sh -c "until [ -f /tmp/setup_done ]; do sleep 5; done" || (echo "Backend setup timeout" && exit 1)
                '''
            }
        }
        
        stage('Prepare Test Environment') {
            steps {
                echo 'Preparing test environment...'
                bat '''
                    REM Check if Composer dependencies are installed
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd composer install --no-interaction --optimize-autoloader --no-dev
                    
                    REM Install dev dependencies (including PHPUnit)
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd composer install --no-interaction --optimize-autoloader
                    
                    REM Create test database schema
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:database:create --env=test --if-not-exists || echo "Test database already exists"
                    
                    REM Run migrations for test database
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:migrations:migrate --env=test --no-interaction || echo "Migrations completed"
                    
                    REM Load fixtures if available
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd php bin/console doctrine:fixtures:load --env=test --no-interaction || echo "No fixtures to load"
                '''
            }
        }
        
        stage('Run PHPUnit Tests') {
            steps {
                echo 'Running PHPUnit tests...'
                bat '''
                    REM Verify PHPUnit installation
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ./vendor/bin/phpunit --version
                    
                    REM Run tests with proper configuration
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ./vendor/bin/phpunit --configuration phpunit.xml.dist --testdox --coverage-text
                '''
            }
        }
        
        stage('Collect Test Results') {
            steps {
                echo 'Collecting test results...'
                bat '''
                    REM Generate JUnit XML report if configured
                    docker-compose -p %COMPOSE_PROJECT_NAME% exec -T backenddd ./vendor/bin/phpunit --configuration phpunit.xml.dist --log-junit test-results.xml || echo "JUnit XML generation failed"
                    
                    REM Copy test results out of container
                    docker-compose -p %COMPOSE_PROJECT_NAME% cp backenddd:/var/www/test-results.xml ./test-results.xml || echo "No test results to copy"
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            bat '''
                REM Show logs for debugging
                docker-compose -p %COMPOSE_PROJECT_NAME% logs backenddd || echo "No logs available"
                
                REM Clean up containers and volumes
                docker-compose -p %COMPOSE_PROJECT_NAME% down --volumes --remove-orphans || echo "Cleanup completed"
            '''
            
            // Archive test results if available
            script {
                if (fileExists('test-results.xml')) {
                    junit 'test-results.xml'
                }
            }
        }
        
        success {
            echo 'All tests passed! ✅'
            script {
                // Send notification if configured
                echo 'Tests completed successfully'
            }
        }
        
        failure {
            echo 'Tests failed ❌'
            bat '''
                REM Show detailed logs for debugging
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=50 backenddd || echo "No logs available"
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=50 postgres || echo "No postgres logs"
                docker-compose -p %COMPOSE_PROJECT_NAME% logs --tail=50 postgres_test || echo "No postgres_test logs"
            '''
        }
        
        unstable {
            echo 'Tests completed with warnings ⚠️'
        }
    }
}