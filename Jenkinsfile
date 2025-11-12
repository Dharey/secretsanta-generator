pipeline {
    agent any

    tools {
        jdk "Java_JDK"
        maven "Java_Maven"
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
        SERVICE_NAME = "secretsanta-generator"
        ORGANIZATION_NAME = "deetechpro"
        DOCKERHUB_USERNAME = "oluwaseyi12"
        REPOSITORY_TAG = "${DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
    }

    stages {
        stage('Code-Compile') {
            steps {
                sh "mvn clean compile"
            }
        }

        stage('Unit Tests') {
            steps {
                sh "mvn test"
            }
        }

        stage('Code-Build') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Docker Build & Push') {
            steps {
                withDockerRegistry([credentialsId: 'DOCKERHUB_USERNAME', url: ""]) {
                    sh "docker build -t ${REPOSITORY_TAG} ."
                    sh "docker push ${REPOSITORY_TAG}"
                }
            }
        }

        stage("Install kubectl") {
            steps {
                sh """
                    curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    ./kubectl version --client
                """
            }
        }

        stage('Docker Image Scan with Trivy') {
            steps {
                sh "trivy image ${REPOSITORY_TAG}"
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to approve the deployment?', ok: 'Yes'
                }
                echo "Deployment Approved"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes']) {
                    script {
                        sh '''
                            envsubst < ${WORKSPACE}/deployment-service.yaml | ./kubectl apply -f -
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<html>
                            <body>
                                <p>Status: SUCCESS</p>
                                <p>Build Number: ${env.BUILD_NUMBER}</p>
                                <p>Check <a href="${env.BUILD_URL}">console output</a>.</p>
                            </body>
                        </html>""",
                to: 'adex0404@gmail.com',
                mimeType: 'text/html'
            )
        }

        failure {
            emailext(
                subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """<html>
                            <body>
                                <p>Status: FAILURE</p>
                                <p>Build Number: ${env.BUILD_NUMBER}</p>
                                <p>Check <a href="${env.BUILD_URL}">console output</a>.</p>
                            </body>
                        </html>""",
                to: 'adex0404@gmail.com',
                mimeType: 'text/html'
            )
        }
    }
}