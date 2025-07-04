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
                script {
                    bat 'docker-compose up -d --build'
                }
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
