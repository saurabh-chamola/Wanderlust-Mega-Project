pipeline {
    agent any

    environment {
        SONAR_HOME = tool 'Sonar'
        DOCKER_REPO = 'your-docker-repo'
    }

    stages {
        stage('Workspace Cleanup') {
            steps { cleanWs() }
        }

        stage('Git: Code Checkout') {
            steps {
                git url: 'https://github.com/saurabh-chamola/Wanderlust-Mega-Project.git', branch: 'main'
            }
        }

        stage('Trivy: Vulnerability Scan') {
            steps {
                script {
                    sh 'trivy fs . > trivy-report.txt'
                    archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
                }
            }
        }

        stage('Docker: Build Images') {
            steps {
                script {
                    dir('backend') {
                        sh 'docker build -t wanderlust-backend-beta:new .'
                    }
                    dir('frontend') {
                        sh 'docker build -t wanderlust-frontend-beta:new .'
                    }
                }
            }
        }

        stage('Run Containers for Testing') {
            steps {
                script {
                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerPass', usernameVariable: 'dockerUser')]) {
                        sh "docker login -u ${dockerUser} -p ${dockerPass}"
                        sh "docker tag wanderlust-frontend-beta:new ${dockerUser}/wanderlust-frontend-beta:${env.BUILD_NUMBER}"
                        sh "docker tag wanderlust-backend-beta:new ${dockerUser}/wanderlust-backend-beta:${env.BUILD_NUMBER}"
                        sh "docker push ${dockerUser}/wanderlust-backend-beta:${env.BUILD_NUMBER}"
                        sh "docker push ${dockerUser}/wanderlust-frontend-beta:${env.BUILD_NUMBER}"
                    }
                }
            }
        }
    }


    post {
        success {
            // Trigger CD Pipeline with BUILD_NUMBER
            build job: 'wanderlust-CD', 
                  parameters: [
                      string(name: 'BUILD_NUMBER', value: "${env.BUILD_NUMBER}")
                  ]
        }

        always {
            emailext attachLog: true,
                    from: 'sourabhchamola5@gmail.com',
                    to: 'sourabhchamola5@gmail.com',
                    subject: "CI/CD Pipeline - ${currentBuild.result}",
                    body: """
                        <html>
                        <body>
                            <h3>CI/CD Pipeline Report</h3>
                            <p><strong>Build Status:</strong> ${currentBuild.result}</p>
                            <p><strong>Build ID:</strong> ${BUILD_ID}</p>
                            <p><strong>Trivy Report:</strong> <a href='${env.BUILD_URL}artifact/trivy-report.txt'>View Report</a></p>
                            <p><strong>OWASP Report:</strong> <a href='${env.BUILD_URL}artifact/owasp-report.xml'>View Report</a></p>
                            <p><strong>SonarQube:</strong> <a href='http://localhost:9000/dashboard?id=nodetodo'>View Dashboard</a></p>
                        </body>
                        </html>
                    """,
                    mimeType: 'text/html'
        }
    }
}
