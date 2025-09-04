pipeline {
    agent any


    environment {
        Git_Credentials_ID = 'Git_Credentials_ID'
        NGINX_SERVER_SSH_KEY ='Nginx_Server_SSHKey' 
        NGINX_SERVER_IP =  '172.31.44.238'   
        SSH_USER = 'ec2-user'
        NGINX_SERVER_PATH = '/usr/share/nginx/html/'            
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm // Fixed typo
            }
        }
        stage('Deploy to Nginx Server') {
            steps {
                sshagent([${Nginx_Server_SSHKey}]) {
                    sh """
                        echo "Deploying to Nginx Server"
                        ssh -o StrictHostKeyChecking=no ${SSH_USER}@${NGINX_SERVER_IP} sudo rm -rf /usr/share/nginx/html/*'
                        scp -o StrictHostKeyChecking=no ${SSH_USER}@${NGINX_SERVER_IP}:/usr/share/nginx/html/
                        echo "Restarting Nginx Server"
                        sleep 10
                        ssh ${SSH_USER}@${NGINX_SERVER_IP} 'sudo systemctl restart nginx'
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