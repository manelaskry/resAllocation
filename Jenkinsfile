pipeline {
    agent any

    triggers {
        cron('0 2 * * *')
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/manelaskry/resAllocation.git'
            }
        }

        stage('Start Containers') {
            steps {
                bat '''
                    docker-compose down --volumes --remove-orphans || exit /b 0
                    docker-compose up -d backenddd
                '''
            }
        }

        stage('Wait for Setup Completion') {
            steps {
                bat '''
                    echo "Waiting for container setup to complete..."
                    set /a attempts=0
                    :wait_for_setup
                    set /a attempts+=1
                    if %attempts% GEQ 60 (
                        echo "Setup timeout after 10 minutes!"
                        docker-compose logs backenddd
                        goto end
                    )
                    docker-compose exec -T backenddd test -f /tmp/setup_done >nul 2>&1 && goto setup_ready
                    echo "Setup in progress... (attempt %attempts%/60)"
                    ping 127.0.0.1 -n 11 > nul
                    goto wait_for_setup

                    :setup_ready
                    echo "Setup completed!"
                    goto end

                    :end
                '''
            }
        }

        stage('Run Backend Tests') {
            steps {
                bat '''
                    echo "Running PHPUnit tests..."
                    docker-compose exec -T backenddd ./vendor/bin/phpunit tests/Entity/
                '''
            }
        }

        stage('Teardown') {
            steps {
                bat 'docker-compose down'
            }
        }

    }
}
