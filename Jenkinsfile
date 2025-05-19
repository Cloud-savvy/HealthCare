pipeline {
    agent any

    environment {
        DOCKER_USER = "bromaaascripts"
        IMAGE = "${DOCKER_USER}/health-care:${BUILD_NUMBER}"
    }

    stages {
        stage('git clone'){
            steps {
                git 'https://github.com/Cloud-savvy/HealthCare'
            }
        }

        stage('Maven build'){
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
    
        stage('Build Docker image'){
            steps {
                sh "docker build -t $IMAGE ."
            }
        }

        stage('Login and push to DockerHub'){
            steps {
                withCredentials([string(credentialsId: 'docker-cred', variable: 'dockerhubpasswd')]) {
                sh "echo ${dockerhubpasswd} | docker login -u ${DOCKER_USER} --password-stdin"
                sh "docker push $IMAGE"
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
            // Proper substitution: Groovy interpolates ${BUILD_NUMBER}, and we escape $ for sed
            sh "sed 's|\\\$BUILD_NUMBER|${BUILD_NUMBER}|g' k8s/deployment-template.yaml > k8s/deployment.yaml"
            // Optional: Show the processed YAML for debugging
            sh 'cat k8s/deployment.yaml'
            // Apply the deployment
            sh 'kubectl apply -f k8s/deployment.yaml'
            // Show Kubernetes resources
            sh 'kubectl get all'
            }
        }
    }
}
