pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"
        DOCKER_USER = "yourdockerusername"
        IMAGE_NAME = "rushi-tech-site"
        TAG = "latest"
        FULL_IMAGE = "${REGISTRY}/${DOCKER_USER}/${IMAGE_NAME}:${TAG}"
        REMOTE_USER = "ec2-user"
        REMOTE_HOST = "your.ec2.ip.address"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-org/rushi-technologies-site.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ."
            }
        }

        stage('DockerHub Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_HUB_USER', passwordVariable: 'DOCKER_HUB_PASS')]) {
                    sh '''
                        echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                        docker push ${FULL_IMAGE}
                    '''
                }
            }
        }

        stage('Remote Deployment via Pull') {
            steps {
                sshagent(['your-ssh-key-id']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no $REMOTE_USER@$REMOTE_HOST "
                            echo \"Logging into DockerHub...\"
                            echo \"$DOCKER_HUB_PASS\" | docker login -u \"$DOCKER_USER\" --password-stdin &&
                            docker stop rushi-container || true &&
                            docker rm rushi-container || true &&
                            docker pull ${FULL_IMAGE} &&
                            docker run -d -p 80:80 --name rushi-container ${FULL_IMAGE}
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            slackSend(channel: '#builds', message: "Job: ${env.JOB_NAME} #${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}", color: "#439FE0")
            office365ConnectorSend webhookUrl: 'https://outlook.office.com/webhook/your-webhook-url', message: "Job: ${env.JOB_NAME} #${env.BUILD_NUMBER} finished with status: ${currentBuild.currentResult}"
            cleanWs()
        }
        success {
            emailext (
                subject: "Jenkins Job SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: '''<p>Build Succeeded!</p>
                         <p><b>Job:</b> ${env.JOB_NAME}</p>
                         <p><b>Build #:</b> ${env.BUILD_NUMBER}</p>
                         <p><b>URL:</b> <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>''',
                to: "your_email@gmail.com"
            )
        }

        failure {
            emailext (
                subject: "Jenkins Job FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: '''<p>Build Failed!</p>
                         <p><b>Job:</b> ${env.JOB_NAME}</p>
                         <p><b>Build #:</b> ${env.BUILD_NUMBER}</p>
                         <p><b>URL:</b> <a href='${env.BUILD_URL}'>${env.BUILD_URL}</a></p>''',
                to: "your_email@gmail.com"
            )
        }

        always {
            cleanWs()
        }
    }
}
