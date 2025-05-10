pipeline {
    agent any

    environment {
        SONAR_HOME= tool "Sonar"
    }

    stages {
        stage('Workspace Cleanup') {
            steps { cleanWs() }
        }

        stage('Trivy: Vulnerability Scan') {
            steps {
                script {
                    sh 'trivy fs --format table --output trivy-report.txt .'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    // sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo -X"
                }
            }
        }
        

        // stage('SonarQube Quality Gates') {
        //     steps {
        //         timeout(time: 2, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ -o ./owasp-report.xml', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/owasp-report.xml'
            }
        }

        stage('Docker Build and Push') {
            steps {
                withCredentials([usernamePassword(credentialsId:"docker-cred",passwordVariable:"dockerPass",usernameVariable:"dockerUser")]){
                    sh "docker login -u ${dockerUser} -p ${dockerPass}"
                    sh "docker build -t ${dockerUser}/node-app-batch-6:${BUILD_ID} ."
                    sh "docker push ${dockerUser}/node-app-batch-6:${BUILD_ID}"
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
    }

    post {
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
@Library('Shared') _
pipeline {
    agent any

    environment {
        SONAR_HOME= tool "Sonar"
        DOCKER_REPO = 'your-docker-repo'
    }

    stages {
        stage('Workspace Cleanup') {
            steps { cleanWs() }
        }

        stage('Trivy: Vulnerability Scan') {
            steps {
                script {
                    sh 'trivy fs --format table --output trivy-report.txt .'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust -X"
                }
            }
        }

        stage('SonarQube Quality Gates') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ -o ./owasp-report.xml', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/owasp-report.xml'
            }
        }

        stage('Docker: Build Images'){
            steps{
                script{
                        dir('backend'){
                            sh "docker build -t wanderlust-backend-beta:new ."
                        }
                        
                        dir('frontend'){
                            sh "docker build -t wanderlust-frontend-beta:new ."
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
        stage('push to docker hub'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId:"docker-cred",passwordVariable:"dockerPass",usernameVariable:"dockerUser")]){
                        sh "docker login -u ${dockerUser} -p ${dockerPass}"
                        sh "docker tag wanderlust-frontend-beta:new ${dockerUser}/wanderlust-frontend-beta:${env.BUILD_NUMBER}"
                        sh "docker tag wanderlust-backend-beta:new  ${dockerUser}/wanderlust-backend-beta:${env.BUILD_NUMBER}"
                        sh "docker push ${dockerUser}/wanderlust-backend-beta:${env.BUILD_NUMBER}"
                        sh "docker push ${dockerUser}/wanderlust-frontend-beta:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

    }

    post {
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
