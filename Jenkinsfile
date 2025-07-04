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
                bat '''
                    docker-compose down || exit /b 0
                    docker-compose up -d --build
                    timeout /t 20
                    
                    echo "=== Debug: Check file structure ==="
                    docker-compose exec -T backend find /var/www -name "*.json" -o -name "*.php" | head -10
                    
                    echo "=== Install and Test ==="
                    docker-compose exec -T backend bash -c "
                        if [ -f /var/www/composer.json ]; then
                            cd /var/www && composer install && ./vendor/bin/phpunit tests/
                        elif [ -f /var/www/backend/composer.json ]; then
                            cd /var/www/backend && composer install && ./vendor/bin/phpunit tests/
                        else
                            echo 'composer.json not found in expected locations'
                            exit 1
                        fi
                    "
                '''
            }
        }
    }
    
    post {
        always {
            bat 'docker-compose down || exit /b 0'
        }
    }
}
