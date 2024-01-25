
pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile"
        DOCKER_IMAGE_NAME = ""
        TRIVY_REPORT_PATH = "./trivy-report.json"
        HTML_REPORT_DIR = "trivy-html-report"
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
                    sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${DOCKER_IMAGE_NAME}"

                    // Convert Trivy JSON report to HTML using trivy2html
                    sh "trivy2html -s -f ${TRIVY_REPORT_PATH} -o ${HTML_REPORT_DIR}/index.html"
                }
            }
        }
    }

    post {
        always {
            script {
                sh "docker rmi ${DOCKER_IMAGE_NAME}"
                archiveArtifacts artifacts: "${TRIVY_REPORT_PATH}", fingerprint: true

                // Publish HTML report
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: "${HTML_REPORT_DIR}", reportFiles: 'index.html', reportName: 'Trivy Vulnerability Scan Report'])
            }
        }
    }
}
