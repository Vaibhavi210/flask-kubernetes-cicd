pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'your-dockerhub-username'
        IMAGE_NAME = 'flask-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Vaibhavi210/flask-kubernetes-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                    echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                    docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:latest
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl set image deployment/flask-app flask-app=$DOCKERHUB_USER/$IMAGE_NAME:latest
                kubectl rollout status deployment/flask-app
                '''
            }
        }
    }
}
