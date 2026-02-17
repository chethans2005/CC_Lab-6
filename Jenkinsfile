pipeline {
    agent any

    stages {

        stage('Docker Login (Prevent Rate Limit)') {
            steps {
                sh '''
                echo "Logging into Docker..."
                docker login -u $DOCKER_USER -p $DOCKER_PASS
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network inspect app-network >/dev/null 2>&1 || docker network create app-network
                
                docker rm -f backend1 backend2 || true
                
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx:alpine
                
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. Load balancer running on port 80.'
        }
        failure {
            echo 'Pipeline failed. Check logs.'
        }
    }
}

