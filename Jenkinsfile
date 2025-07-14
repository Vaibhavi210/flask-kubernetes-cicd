pipeline {
    agent any
    
    environment {
        DOCKERHUB_USER = 'vaibhavi210'
        IMAGE_NAME = 'flask-app'
        AWS_DEFAULT_REGION = 'ap-south-1'  // ✅ Update to your region
        EKS_CLUSTER_NAME = 'devops-cluster'  // ✅ Update to your cluster name
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out source code..."
                git branch: 'main', url: 'https://github.com/Vaibhavi210/flask-kubernetes-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🏗️ Building Docker image..."
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

        stage('Install kubectl') {
            steps {
                echo "📦 Installing kubectl..."
                script {
                    try {
                        sh '''
                            # Check if kubectl is already installed
                            if ! command -v kubectl &> /dev/null; then
                                echo "Installing kubectl..."
                                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                                chmod +x kubectl
                                sudo mv kubectl /usr/local/bin/
                            else
                                echo "kubectl already installed"
                            fi
                            kubectl version --client
                        '''
                    } catch (Exception e) {
                        error "❌ kubectl installation failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo "☸️ Deploying to EKS cluster..."
                withCredentials([
                    aws(credentialsId: 'aws-creds', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    script {
                        try {
                            sh '''
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
                                echo "📊 Deployment Status:"
                                kubectl get deployment flask-app
                                kubectl get service flask-app
                                kubectl get pods -l app=flask-app
                                
                                # Get LoadBalancer URL
                                echo "🌐 Application URL:"
                                kubectl get service flask-app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' && echo ":5000"
                                
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
            script {
                try {
                    sh '''
                        # Clean up Docker images to save space
                        docker rmi $DOCKERHUB_USER/$IMAGE_NAME:latest || true
                        docker rmi $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER || true
                        docker rmi $IMAGE_NAME:latest || true
                        docker system prune -f || true
                    '''
                } catch (Exception e) {
                    echo "⚠️ Cleanup warning: ${e.getMessage()}"
                }
            }
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            echo "🌐 Your Flask app should be accessible via the LoadBalancer URL shown above"
        }
        failure {
            echo "💥 Pipeline failed! Check the logs above for details."
        }
    }
}