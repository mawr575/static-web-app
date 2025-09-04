pipeline {
    agent any

    environment {
        Git_Credentials_ID = 'Git_Credentials'
        NGINX_SERVER_SSH_KEY = 'Nginx_Server_SSHKey'
        NGINX_SERVER_IP = '172.31.44.238'
        SSH_USER = 'ec2-user'
        NGINX_SERVER_PATH = '/usr/share/nginx/html/'
    }

    trigger {
        githubPush()
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Deploy to Nginx Server') {
            steps {
                sshagent([env.NGINX_SERVER_SSH_KEY]) {
                    sh """
                        echo "Deploying to Nginx Server"
                        ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.NGINX_SERVER_IP} "sudo rm -rf ${env.NGINX_SERVER_PATH}*"
                        scp -o StrictHostKeyChecking=no -r * ${env.SSH_USER}@${env.NGINX_SERVER_IP}:${env.NGINX_SERVER_PATH}
                        echo "Restarting Nginx Server"
                        sleep 10
                        ssh ${env.SSH_USER}@${env.NGINX_SERVER_IP} "sudo systemctl restart nginx"
                    """
                }
            }
        }
    }
}