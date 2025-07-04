pipeline {
    agent any
    
    triggers {
        cron('0 2 * * *')
    }

    stages {
        stage('Test') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
                
                bat '''
                    docker-compose up -d backenddd
                    
                    REM Wait for setup to complete (check for setup_done marker)
                    :wait_setup
                    docker-compose exec -T backenddd test -f /tmp/setup_done && goto setup_complete
                    echo "Waiting for container setup to complete..."
                    ping 127.0.0.1 -n 11 > nul
                    goto wait_setup
                    
                    :setup_complete
                    echo "Container setup completed!"
                    
                    REM Now run tests
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                    
                    docker-compose down
                '''
            }
        }
    }
}
