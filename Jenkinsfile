pipeline {
    agent any
    
    environment {
        DOCKERHUB_USER = 'vaibhavi210'
        IMAGE_NAME = 'flask-app'
        AWS_DEFAULT_REGION = 'ap-south-1'  
        EKS_CLUSTER_NAME = 'devops-cluster'  
    }

    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Checking out source code..."
                git branch: 'main', url: 'https://github.com/Vaibhavi210/flask-kubernetes-cicd.git'
            }
        }

        stage('Fix Docker Permissions & Build') {
            steps {
                echo "🔧 Fixing Docker permissions and building image..."
                script {
                    try {
                        sh '''
                            # Try to fix Docker permissions
                            sudo chown root:docker /var/run/docker.sock || true
                            sudo chmod 666 /var/run/docker.sock || true
                            
                            # Build Docker image
                            docker build -t $IMAGE_NAME:latest .
                            docker images | grep $IMAGE_NAME
                        '''
                        echo " Docker image built successfully"
                    } catch (Exception e) {
                        error " Docker build failed: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo " Pushing image to DockerHub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        try {
                            sh '''
                                echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                                docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:latest
                                docker tag $IMAGE_NAME:latest $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                                docker push $DOCKERHUB_USER/$IMAGE_NAME:$BUILD_NUMBER
                                echo "Image pushed successfully"
                            '''
                        } catch (Exception e) {
                            error " Docker push failed: ${e.getMessage()}"
                        }
                    }
                }
            }
        }

        stage('Install kubectl') {
    steps {
        echo ' Installing kubectl if not present...'
        sh '''
            if ! command -v kubectl &> /dev/null; then
                echo "Installing kubectl..."
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x kubectl
                mkdir -p $HOME/bin
                mv kubectl $HOME/bin/kubectl
                export PATH=$HOME/bin:$PATH
            else
                echo "kubectl already installed"
            fi
            kubectl version --client
        '''
    }
}

stage('Deploy to EKS') {
    steps {
        echo " Deploying to EKS cluster..."
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
            credentialsId: 'aws-creds'
        ]]) {
            script {
                sh '''
                    if ! command -v kubectl &> /dev/null; then
                echo "Installing kubectl..."
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x kubectl
                mkdir -p $HOME/bin
                mv kubectl $HOME/bin/kubectl
                export PATH=$HOME/bin:$PATH
            else
                echo " kubectl already installed"
            fi
            kubectl version --client

                    # Use safe temp home to avoid kubeconfig corruption
                    export HOME=/tmp
                    mkdir -p $HOME/.kube

                    echo "🔗 Updating kubeconfig for EKS cluster..."
                    aws eks update-kubeconfig --region $AWS_DEFAULT_REGION --name $EKS_CLUSTER_NAME

                    echo "🔍 Checking if deployment exists..."
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

                    echo "📊 Deployment Status:"
                    kubectl get deployment flask-app
                    kubectl get service flask-app
                    kubectl get pods -l app=flask-app

                    echo "🌐 Application URL:"
                    kubectl get service flask-app -o jsonpath='{.status.loadBalancer.ingress[0].hostname}' && echo ":5000"
                '''
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
                        docker container prune -f || true
                    '''
                } catch (Exception e) {
                    echo "⚠️ Cleanup warning: ${e.getMessage()}"
                }
            }
        }
        success {
            echo "🎉 Pipeline completed successfully!"
            echo " Your Flask app should be accessible via the LoadBalancer URL shown above"
        }
        failure {
            echo " Pipeline failed! Check the logs above for details."
        }
    }
}