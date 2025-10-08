pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build (Maven)') {
            steps {
                sh 'docker run --rm -v "$PWD":/work -w /work maven:3.8.8-openjdk-17 mvn -B clean package -DskipTests'
            }
        }
        stage('Build Docker Images') {
            steps {
                sh '''
                  docker build -t myapp:${BUILD_NUMBER} -f Dockerfile.app .
                  docker build -t mynginx:${BUILD_NUMBER} -f Dockerfile.nginx .
                '''
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                  # ensure network exists
                  docker network inspect jenkins-net >/dev/null 2>&1 || docker network create jenkins-net

                  # stop previous containers if any
                  docker rm -f myapp || true
                  docker rm -f mynginx || true

                  # run app and nginx
                  docker run -d --name myapp --network jenkins-net myapp:${BUILD_NUMBER}
                  docker run -d --name mynginx --network jenkins-net -p 80:80 mynginx:${BUILD_NUMBER}
                '''
            }
        }
        stage('Smoke Test') {
            steps {
                sh 'sleep 5'
                sh 'curl -f http://localhost/ || (echo "Smoke test failed"; exit 1)'
            }
        }
    }
    post {
        success {
            echo "Deployed myapp:${BUILD_NUMBER} behind nginx"
        }
        failure {
            echo "Build or deploy failed"
        }
    }
}
