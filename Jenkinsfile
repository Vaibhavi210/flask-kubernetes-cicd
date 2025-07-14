pipeline {
    agent none
    
    environment {
        DOCKERHUB_USER = 'vaibhavi210'
        IMAGE_NAME = 'flask-app'
        AWS_DEFAULT_REGION = 'ap-south-1' 
        EKS_CLUSTER_NAME = 'devops-cluster'  
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                echo "🔄 Checking out source code..."
                git branch: 'main', url: 'https://github.com/Vaibhavi210/flask-kubernetes-cicd.git'
                stash includes: '**', name: 'source'
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "🏗️ Building Docker image..."
                unstash 'source'
                script {
                    try {
                        sh 'docker build -t $IMAGE_NAME:latest .'
                        sh 'docker images | grep $IMAGE_NAME'
                        echo "✅ Docker image built successfully"
                    } catch (Exception e) {
                        error "❌ Docker build failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "🚀 Pushing image to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        try {
                            sh '''
                                echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                                docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:latest
                                docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                                docker push $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                echo "✅ Image pushed successfully"
                            '''
                        } catch (Exception e) {
                            error "❌ Docker push failed: ${e.getMessage()}"
                        }
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            agent {
                docker {
                    image 'amazon/aws-cli:latest'
                    args '--entrypoint=""'
                }
            }
            steps {
                echo "☸️ Deploying to EKS cluster..."
                withCredentials([
                    aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        try {
                            sh '''
                                # Install kubectl
                                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                                chmod +x kubectl
                                mv kubectl /usr/local/bin/
                                
                                # Configure kubectl for EKS
                                aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $EKS_CLUSTER_NAME
                                
                                # Verify connection
                                kubectl get nodes
                                
                                # Check if deployment exists
                                if kubectl get deployment flask-app > /dev/null 2>&1; then
                                    echo "📝 Updating existing deployment..."
                                    kubectl set image deployment/flask-app flask-app=$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                    kubectl rollout status deployment/flask-app --timeout=300s
                                else
                                    echo "🆕 Creating new deployment..."
                                    kubectl create deployment flask-app --image=$DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                    kubectl expose deployment flask-app --port=5000 --type=LoadBalancer
                                    kubectl rollout status deployment/flask-app --timeout=300s
                                fi
                                
                                # Get deployment info
                                kubectl get deployment flask-app
                                kubectl get service flask-app
                                kubectl get pods -l app=flask-app
                                
                                echo "✅ Deployment completed successfully"
                            '''
                        } catch (Exception e) {
                            error "❌ EKS deployment failed: ${e.getMessage()}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up..."
            node {
                // Clean up Docker images to save space
                sh '''
                    docker rmi $DOCKERHUB_USER/$IMAGE_NAME:latest || true
                    docker rmi $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER || true
                    docker rmi $IMAGE_NAME:latest || true
                    docker system prune -f || true
                '''
            }
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            // You can add notifications here (Slack, email, etc.)
        }
        failure {
            echo "💥 Pipeline failed!"
            // You can add failure notifications here
        }
    }
}