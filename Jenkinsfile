pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'cd backend && composer install'
            }
        }

        stage('Run PHPUnit Tests') {
            steps {
                sh 'cd backend && vendor/bin/phpunit tests/'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
