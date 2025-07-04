pipeline {
    agent any

    stages {
        stage('Debug Container') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
                
                bat '''
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker-compose up -d backenddd
                    ping 127.0.0.1 -n 31 > nul
                    
                    echo "=== CONTAINER DEBUG INFO ==="
                    docker-compose exec -T backenddd pwd
                    docker-compose exec -T backenddd ls -la
                    docker-compose exec -T backenddd ls -la backend/ || echo "No backend dir"
                    docker-compose exec -T backenddd cat composer.json || echo "No composer.json"
                    docker-compose exec -T backenddd which composer || echo "No composer"
                    docker-compose exec -T backenddd php -v || echo "No PHP"
                    
                    echo "=== DOCKER-COMPOSE CONFIG ==="
                    docker-compose config
                    
                    docker-compose down
                '''
            }
        }
    }
}
