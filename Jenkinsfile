pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm // Fixed typo
            }
        }
        stage('Deploy to Nginx Server') {
            steps {
                sshagent(['Nginx_Server_SSHKey']) {
                    sh """
                        echo "Deploying to Nginx Server"
                        ssh -o StrictHostKeyChecking=no ec2-user@172.31.44.238 'sudo rm -rf /usr/share/nginx/html/*'
                        scp -o StrictHostKeyChecking=no -r * ec2-user@172.31.44.238:/usr/share/nginx/html/
                        echo "Restarting Nginx Server"
                        sleep 10
                        ssh ec2-user@172.31.44.238 'sudo systemctl restart nginx'
                    """
                }
            }
        }
    }
    post {
        success {
            emailext(
                subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build SUCCESS",
                body: "Build SUCCESS. Please check the console output at ${env.BUILD_URL}",
                to: 'riyaz.awr57565@gmail.com',
                mimeType: 'text/html'
            )
        }
        failure {
            emailext(
                subject: "${env.JOB_NAME} - ${env.BUILD_NUMBER} - Build FAILED",
                body: "Build FAILED. Please check the console output at ${env.BUILD_URL}",
                to: 'riyaz.awr57565@gmail.com',
                mimeType: 'text/html'
            )
        }
    }
}