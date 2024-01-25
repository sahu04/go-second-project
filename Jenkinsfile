pipeline {
    agent any
    
    environment {
        DOCKERFILE_PATH = "./Dockerfile"
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/sahu04/go-second-project.git'
            }
        }
        
    stage('Build Docker Image') {
    steps {
        script {
            // Use double quotes and escape inner double quotes
            sh "dockerImageName=\$(awk \"NR==1 {print \$2}\" $DOCKERFILE_PATH)"
            sh "docker build -t $dockerImageName -f $DOCKERFILE_PATH ."
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
                    sh "docker push $dockerImageName"
                }
            }
        }
    }
    
    post {
        always {
            script {
                sh "docker rmi $dockerImageName"
            }
        }
    }
}
