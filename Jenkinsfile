pipeline {
    agent any
    stages {
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yml'
            }
        }
    }
}
