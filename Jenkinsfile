pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "pawan16/jenkins-deployment"
        TAG = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/pawan16/demo.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:$TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub_credentials',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh """
                    echo "$PASS" | docker login -u "$USER" --password-stdin
                    docker push $DOCKER_IMAGE:$TAG
                    """
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['ssh_deploy_key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no user@your-server-ip "
                        docker pull $DOCKER_IMAGE:$TAG &&
                        docker stop java-demo || true &&
                        docker rm java-demo || true &&
                        docker run -d --name java-demo -p 8080:8080 $DOCKER_IMAGE:$TAG
                    "
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment success!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
