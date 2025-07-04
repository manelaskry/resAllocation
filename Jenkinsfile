pipeline {
    agent any

    triggers {
        cron('0 2 * * *')
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
            }
        }

        stage('Start Containers') {
            steps {
                echo 'Starting containers...'
            }
        }

        stage('Wait for Setup Completion') {
            steps {
                echo 'Waiting for setup to complete...'
            }
        }

        stage('Run Backend Tests') {
            steps {
                echo 'Running backend tests...'
            }
        }

        stage('Teardown') {
            steps {
                echo 'Tearing down containers...'
            }
        }

    }
}
