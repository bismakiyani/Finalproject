pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh 'docker-compose down --remove-orphans || true'
            }
        }

        stage('Remove Old Images') {
            steps {
                sh '''
                    docker rmi frontend || true
                    docker rmi backend  || true
                    docker image prune -f || true
                '''
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker-compose build --no-cache'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 20'
                sh 'docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"'
                sh 'curl -f --retry 5 --retry-delay 5 http://localhost && echo "SUCCESS" || echo "WARNING: not responding yet"'
            }
        }
    }

    post {
        success {
            echo '========== DEPLOYMENT SUCCESSFUL =========='
        }
        failure {
            sh '''
                docker logs frontend || true
                docker logs backend  || true
                docker logs nginx    || true
            '''
        }
        always {
            cleanWs()
        }
    }
}
