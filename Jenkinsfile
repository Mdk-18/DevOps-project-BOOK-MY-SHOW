pipeline {  
agent any  
tools {  
jdk 'jdk17'  
nodejs 'node23'  
}  
    environment {  
        DOCKER_IMAGE = 'mdk18/bms:latest'  
        EKS_CLUSTER_NAME = 'bms-eks'  
        AWS_REGION = 'us-east-1'  
    }  
  
    stages {  
        stage('Clean Workspace') {  
            steps {  
                cleanWs()  
            }  
        }  
  
        stage('Checkout from Git') {  
            steps {  
               git branch: 'main', url: 'https://github.com/KastroVKiran/Book-My-Show.git’ 
               sh 'ls -la'  // Verify files after checkout  
            }  
        }  
  
        stage('Install Dependencies') {  
            steps {  
                sh '''  
                cd bookmyshow-app  
                ls -la  # Verify package.json exists  
                if [ -f package.json ]; then  
                    rm -rf node_modules package-lock.json  # Remove old dependencies  
                    npm install  # Install fresh dependencies  
                else  
                    echo "Error: package.json not found in bookmyshow-app!"  
                    exit 1  
                fi  
                '''  
            }  
        }  
  
        stage('Docker Build & Push') {  
            steps {  
                script {  
                   withDockerRegistry(credentialsId: 'dockerhub-creds') {  
                        sh '''  
                        docker build -t mdk18/bms:latest -f bookmyshow-app/Dockerfile  
bookmyshow-app  
                        docker push mdk18/bms:latest  
                    '''  
                        }  
                }  
            }  
        }  
  
       stage('Deploy to EKS Cluster') {  
    steps {  
        script {  
            withCredentials([[  
                $class: 'AmazonWebServicesCredentialsBinding',  
                credentialsId: 'aws-cred'  
            ]]) {  
                sh '''  
                    echo "Verifying AWS credentials..."  
                    aws sts get-caller-identity  
                    aws eks update-kubeconfig --name bms-eks --region us-east-1  
                    kubectl apply -f deployment.yml  
                    kubectl apply -f service.yml  
                '''  
            }  
        }  
    }  
}  
  
    }  
  
    post {  
        success {  
            echo "Deployment completed successfully!"  
        }  
        failure {  
            echo "Deployment failed! Please check the Jenkins logs."  
        }  
        always {  
            echo "Pipeline execution finished."  
        }  
    }  
} 
