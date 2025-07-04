pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Check Docker') {
            steps {
                bat 'docker --version'
                bat 'docker-compose --version'
                bat 'dir'
            }
        }
        
        stage('Check Docker Compose File') {
            steps {
                bat 'type docker-compose.yml'
            }
        }
        
        stage('Docker Build') {
            steps {
                bat 'docker-compose down'
                bat 'docker-compose up -d --build'
                bat 'docker-compose ps'
            }
        }
        
        stage('Test Docker Connection') {
            steps {
                bat 'docker-compose exec -T backend php --version'
            }
        }
    }
    
    post {
        always {
            echo 'Debug terminé!'
            bat 'docker-compose down'
        }
    }
}
