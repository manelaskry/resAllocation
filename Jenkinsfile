pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Docker Build') {
            steps {
                bat '''
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker container prune -f || exit /b 0
                    docker-compose up -d --build
                '''
            }
        }
        
        stage('Install Dependencies') {
            steps {
                bat 'docker-compose exec -T backend composer install --no-interaction'
            }
        }
        
        stage('Run Project Tests') {
            steps {
                bat 'docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/ProjectTest.php --testdox'
            }
        }
        
        stage('Run User Tests') {
            steps {
                bat 'docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/UserTest.php --testdox'
            }
        }
    }
    
    post {
        always {
            echo 'Tests terminés!'
            bat 'docker-compose down --volumes --remove-orphans || exit /b 0'
        }
        success {
            echo 'Tous les tests sont passés ✅'
        }
        failure {
            echo 'Certains tests ont échoué ❌'
        }
    }
}
