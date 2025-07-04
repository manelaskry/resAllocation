pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Start Services') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat '''
                        docker-compose down || exit /b 0
                        docker-compose up -d
                        timeout /t 30 /nobreak > nul
                        docker-compose ps
                    '''
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat 'docker-compose exec -T backend composer install --no-interaction'
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    bat '''
                        docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/ProjectTest.php --testdox
                        docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/UserTest.php --testdox
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Tests terminés!'
            bat 'docker-compose down || exit /b 0'
        }
        success {
            echo 'Tous les tests sont passés ✅'
        }
        failure {
            echo 'Certains tests ont échoué ❌'
        }
    }
}
