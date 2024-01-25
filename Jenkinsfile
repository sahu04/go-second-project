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
                    DOCKER_IMAGE_NAME = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    sh "docker build -t ${DOCKER_IMAGE_NAME} -f ${DOCKERFILE_PATH} ."
                }
            }
        }

        stage('Vulnerability Scan - Docker Trivy') {
            steps {
                script {
                    echo "Running Trivy scan for image: ${DOCKER_IMAGE_NAME}"
                    sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o trivy-report.json ${DOCKER_IMAGE_NAME}"
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
        success {
            script {
                archiveArtifacts artifacts: 'trivy-report.json', fingerprint: true
            }
        }
    }
}

