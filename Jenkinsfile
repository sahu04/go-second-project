pipeline {
    agent any
    
    environment {
        // Define any environment variables you may need
        DOCKER_IMAGE_NAME = "tomcat"
        DOCKERFILE_PATH = "./Dockerfile"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Checkout your source code from version control
                git 'https://github.com/sahu04/go-second-project.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t $DOCKER_IMAGE_NAME -f $DOCKERFILE_PATH ."
                }
            }
        }
            stage('Vulnerability Scan - Docker Trivy') {
      steps {
        script {
            echo "Running Trivy scan for image: $dockerImageName"
            withCredentials([string(credentialsId: 'trivy_github_token', variable: 'TOKEN')]) {
                sh "sed -i 's#token_github#${TOKEN}#g' trivy-image-scan.sh"
                sh "sudo trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o trivy-report.json \"$dockerImageName\""
            }
            echo "Trivy scan completed"
        }
    }
}

        stage('Push Docker Image') {
            steps {
                script {
                    // Optionally, push the Docker image to a registry
                    sh "docker push $DOCKER_IMAGE_NAME"
                }
            }
        }
    }
    
    post {
        always {
            // Clean up any resources or perform additional steps
            script {
                // Optionally, remove the locally built image
                sh "docker rmi $DOCKER_IMAGE_NAME"
            }
        }
    }
}
