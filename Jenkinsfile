pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Docker Cleanup') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    bat '''
                        docker-compose down --volumes --remove-orphans || exit /b 0
                        docker system prune -f || exit /b 0
                        docker network prune -f || exit /b 0
                    '''
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    bat '''
                        echo "Starting Docker Compose build..."
                        docker-compose up -d --build --force-recreate
                        echo "Checking container status..."
                        docker-compose ps
                    '''
                }
            }
        }
        
        stage('Install Composer') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat '''
                        echo "Installing Composer..."
                        docker-compose exec -T backend php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
                        docker-compose exec -T backend php composer-setup.php --install-dir=/usr/local/bin --filename=composer
                        docker-compose exec -T backend rm composer-setup.php
                        docker-compose exec -T backend composer --version
                    '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat 'docker-compose exec -T backend composer install --no-interaction --prefer-dist'
                }
            }
        }
        
        stage('Run Project Tests') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat 'docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/ProjectTest.php --testdox'
                }
            }
        }
        
        stage('Run User Tests') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat 'docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/UserTest.php --testdox'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Tests terminés!'
            timeout(time: 2, unit: 'MINUTES') {
                bat '''
                    docker-compose logs --tail=50 || exit /b 0
                    docker-compose down --volumes --remove-orphans || exit /b 0
                '''
            }
        }
        success {
            echo 'Tous les tests sont passés ✅'
        }
        failure {
            echo 'Certains tests ont échoué ❌'
            bat '''
                echo "Container logs:"
                docker-compose logs || exit /b 0
            '''
        }
    }
}
