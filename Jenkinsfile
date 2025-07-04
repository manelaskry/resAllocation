pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Setup & Test') {
            steps {
                sh '''
                    docker-compose up -d --build
                    docker-compose exec -T backend composer install --no-interaction
                    docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/ProjectTest.php --testdox
                    docker-compose exec -T backend ./vendor/bin/phpunit tests/Entity/UserTest.php --testdox
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Tests terminés!'
            sh 'docker-compose down'
        }
        success {
            echo 'Tous les tests sont passés ✅'
        }
        failure {
            echo 'Certains tests ont échoué ❌'
        }
    }
}
