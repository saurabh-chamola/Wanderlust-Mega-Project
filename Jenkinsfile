pipeline {
    agent any

    environment {
        github_url = 'https://github.com/saurabh-chamola/kubernetes.git'
        docker_username = 'saurabh0010'
        docker_repo = "${docker_username}"
    }

    stages {
        stage('Workspace Cleanup') {
            steps {
                cleanWs()
            }
        }

        stage('Git: Checkout Application Code') {
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
                sh "docker buildx build --load -t ${docker_repo}/wanderlust-backend-beta:${env.BUILD_NUMBER} ."
            }
            dir('frontend') {
                sh "docker buildx build --load -t ${docker_repo}/wanderlust-frontend-beta:${env.BUILD_NUMBER} ."
            }
        }
    }
}




        stage('Run Containers for Testing') {
            steps {
                script {
                    // sh 'docker-compose down || true'
                    // sh 'docker-compose up -d'
                      echo '🔧 Hello from Jenkins!'
                }
            }
        }

        stage('Docker: Push Images to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', usernameVariable: 'dockerUser', passwordVariable: 'dockerPass')]) {
                        sh "docker login -u ${dockerUser} -p ${dockerPass}"
                        sh "docker push ${docker_repo}/wanderlust-backend-beta:${env.BUILD_NUMBER}"
                        sh "docker push ${docker_repo}/wanderlust-frontend-beta:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Git: Checkout Kubernetes Repo') {
            steps {
                git url: "${github_url}", branch: 'main'
            }
        }

        stage('CD: Update Kubernetes YAML Files') {
            steps {
                script {
                    dir('/var/lib/jenkins/workspace/wanderlust') {
                        sh "sed -i 's|${docker_username}/wanderlust-backend-beta:.*|${docker_username}/wanderlust-backend-beta:${env.BUILD_NUMBER}|' kubernetes/backend.yaml"
                    }
                    dir('/var/lib/jenkins/workspace/wanderlust') {
                        sh "sed -i 's|${docker_username}/wanderlust-frontend-beta:.*|${docker_username}/wanderlust-frontend-beta:${env.BUILD_NUMBER}|' kubernetes/frontend.yaml"
                    }
                }
            }
        }

        stage("Git: Commit and Push YAML Updates") {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'github-cred', gitToolName: 'Default')]) {
                        sh '''
                            git config --global user.email "ci@wanderlust.com"
                            git config --global user.name "CI Bot"

                            echo "Adding changes..."
                            git add .

                            echo "Committing changes..."
                            git commit -m "CI/CD: Updated image versions to BUILD_NUMBER=$BUILD_NUMBER" || echo "No changes to commit"

                            echo "Pushing changes to GitHub..."
                            git push https://github.com/saurabh-chamola/kubernetes.git main
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            emailext attachLog: true,
                    to: 'sourabhchamola5@gmail.com',
                    subject: "✅ CI/CD Pipeline Success - Build ${env.BUILD_NUMBER}",
                    body: """
                        <html>
                        <body>
                            <h2 style='color:green;'>CI/CD Pipeline Completed Successfully</h2>
                            <p><b>Build ID:</b> ${BUILD_ID}</p>
                            <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                            <p><b>Trivy Report:</b> <a href='${env.BUILD_URL}artifact/trivy-report.txt'>View Trivy Report</a></p>
                        </body>
                        </html>
                    """,
                    mimeType: 'text/html'
        }

        failure {
            emailext attachLog: true,
                    to: 'sourabhchamola5@gmail.com',
                    subject: "❌ CI/CD Pipeline FAILED - Build ${env.BUILD_NUMBER}",
                    body: """
                        <html>
                        <body>
                            <h2 style='color:red;'>CI/CD Pipeline Failed</h2>
                            <p><b>Build ID:</b> ${BUILD_ID}</p>
                            <p><b>Build URL:</b> <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
                        </body>
                        </html>
                    """,
                    mimeType: 'text/html'
        }
    }
}
