pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile" // Dockerfile path 
        TRIVY_REPORT_PATH = "trivy-scan-report.json"
        DOCKLE_REPORT_PATH = "dockle-scan-report.json"
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
                    sh 'sudo chmod o+w /usr/local/bin'
                    sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3'
                    sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                    sh 'trivy --version'
                }
            }
        }
        stage('Install Dockle') {
            steps {
                 script {
                    sh 'curl -LO https://github.com/goodwithtech/dockle/releases/download/v0.4.13/dockle_0.4.13_Linux-64bit.tar.gz'
                    sh 'tar -xzf dockle_0.4.13_Linux-64bit.tar.gz'
                    sh 'sudo mv dockle /usr/local/bin/'
                    sh 'dockle --version'
                }
            }
        }
        stage('Scan image for vulnerabilities') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Trivy scan for image: ${dockerImageName}"
                    // sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                    sh "trivy  --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                }
            }
        }
        stage('Security Scan - Dockle') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Dockle scan for image: ${dockerImageName}"
                     // sh "export DOCKLE_HOST='unix:///var/run/docker.sock' && dockle -f json -o ${DOCKLE_REPORT_PATH}  --exit-code 1 --exit-level fatal ${dockerImageName}"
                     sh "dockle -f json -o ${DOCKLE_REPORT_PATH}  --exit-code 1 --exit-level fatal ${dockerImageName}"
                }
            }
        }
    } 

    post {
    always {
        script {
            archiveArtifacts artifacts: "${TRIVY_REPORT_PATH}, ${DOCKLE_REPORT_PATH}", followSymlinks: false
            deleteDir() // Clean the workspace
            }
        }
    }
} 
