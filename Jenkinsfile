pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("zubair11089/personal-portfolio:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' 
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "ZubairTerminal", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull zubair11089/personal-portfolio:${env.BUILD_ID}
                                        docker stop personal-portfolio-container || true
                                        docker rm personal-portfolio-container || true
                                        docker run -d --name personal-portfolio-container -p 80:80 zubair11089/personal-portfolio:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )

                    // Check if deployment is successful
                    boolean isDeploymentSuccessful = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://13.51.242.92:80', returnStdout: true).trim() == '200'

                    if (!isDeploymentSuccessful) {
                        // Rollback to the previous version
                        def previousSuccessfulTag = readFile('previous_successful_tag.txt').trim()
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "ZubairTerminal",
                                    transfers: [sshTransfer(
                                        execCommand: """
                                            docker pull zubair11089/personal-portfolio:${previousSuccessfulTag}
                                            docker stop personal-portfolio-container || true
                                            docker rm personal-portfolio-container || true
                                            docker run -d --name personal-portfolio-container -p 80:80 zubair11089/personal-portfolio:${previousSuccessfulTag}
                                        """
                                    )]
                                )
                            ]
                        )
                    } else {
                        // Update the last successful tag
                        writeFile file: 'previous_successful_tag.txt', text: "${env.BUILD_ID}"
                    }
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'muaviaa099@gmail.com',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}
