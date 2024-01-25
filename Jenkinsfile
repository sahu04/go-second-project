pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile"
        DOCKER_IMAGE_NAME = ""
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
                    // Set dockerImageName as an environment variable
                    DOCKER_IMAGE_NAME = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    sh "docker build -t ${DOCKER_IMAGE_NAME} -f ${DOCKERFILE_PATH} ."
                }
            }
        }

        stage('Vulnerability Scan - Docker Trivy') {
            steps {
                script {
                    echo "Running Trivy scan for image: ${DOCKER_IMAGE_NAME}"
                    withCredentials([string(credentialsId: 'trivy_github_token', variable: 'TOKEN')]) {
                        script {
                            sh "sed -i 's#token_github#${TOKEN}#g' trivy_dockle.sh"
                            sh "sudo trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o trivy-report.json ${DOCKER_IMAGE_NAME}"
                        }
                    }
                    echo "Trivy scan completed"
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker rmi ${DOCKER_IMAGE_NAME}"
            }
        }
    }
}
