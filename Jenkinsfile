pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }
        
        stage('Test') {
            steps {
                bat '''
                    docker-compose up -d backend
                    timeout /t 30
                    docker-compose exec -T backend composer install
                    docker-compose exec -T backend ./vendor/bin/phpunit tests/
                    docker-compose down
                '''
            }
        }
    }
}
