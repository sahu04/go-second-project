pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile"
        TRIVY_REPORT_PATH = "trivy-scan-report.json"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/sahu04/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    sh "docker build -t ${dockerImageName} -f ${DOCKERFILE_PATH} ."
                    
                    // Continue using dockerImageName directly in the subsequent steps
                    echo "Docker image name: ${dockerImageName}"
                }
            }
        }

           stage('Install Trivy') {
            steps {
                script {
                    sh """
                        wget https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz
                        tar xvf trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz
                        sudo mv trivy /usr/local/bin/trivy
                    """
                    sh 'trivy --version'
                }
            }
        }
        stage('Vulnerability Scan - Docker Trivy') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Trivy scan for image: ${dockerImageName}"
                    sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                }
            }
        }
    }

    post {
        always {
            script {
                // Note: You can't remove the docker image name here, as it's needed for cleanup
                sh "docker rmi ${dockerImageName}"
                archiveArtifacts artifacts: 'trivy-scan-report.json', followSymlinks: false
            }
        }
    }
}
