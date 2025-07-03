pipeline {
    agent any

    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Backend - Install Dependencies') {
            steps {
                dir('backend') {
                    sh 'composer install'
                }
            }
        }
        
        stage('Backend - Run PHPUnit Tests') {
            steps {
                dir('backend') {
                    sh 'vendor/bin/phpunit tests/'
                }
            }
        }
    }
    
    post {
        always {
            echo 'Tests terminés!'
        }
        success {
            echo 'Tous les tests sont passés ✅'
        }
        failure {
            echo 'Certains tests ont échoué ❌'
        }
    }
}