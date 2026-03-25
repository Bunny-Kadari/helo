pipeline {
    agent any

    environment {
        DOCKER_USER = "bunnykadari"
        IMAGE_NAME = "helo"
        DOCKER_IMAGE = "${DOCKER_USER}/${IMAGE_NAME}"
        CONTAINER_NAME = "helo-container"
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/Bunny-Kadari/helo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Run Container (Optional)') {
            steps {
                sh '''
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                docker run -d -p 5000:5000 --name $CONTAINER_NAME $DOCKER_IMAGE
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl delete deployment helo --ignore-not-found
                kubectl create deployment helo --image=$DOCKER_IMAGE
                kubectl expose deployment helo --type=NodePort --port=5000 --ignore-not-found
                '''
            }
        }
    }
}
