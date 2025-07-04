pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/yourusername/your-repo.git'
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
