pipeline {
    agent any

    environment {
        SSH_KEY = credentials('ec2-ssh-key')  // Add your SSH private key in Jenkins credentials
        EC2_IP = "18.215.172.59"
        REMOTE_DIR = "/var/www/nodeapp"
        SSH_USER = "ubuntu"  // Assuming it's an Ubuntu AMI
    }

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/saritaharihar/PetClinic.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Lint & Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh '''
                        tar -czf app.tar.gz .next package.json node_modules public
                        scp -o StrictHostKeyChecking=no app.tar.gz $SSH_USER@$EC2_IP:/tmp/
                        ssh -o StrictHostKeyChecking=no $SSH_USER@$EC2_IP << EOF
                          mkdir -p $REMOTE_DIR
                          tar -xzf /tmp/app.tar.gz -C $REMOTE_DIR
                          cd $REMOTE_DIR
                          npm install --omit=dev
                          pm2 restart all || pm2 start npm --name "nextjs-app" -- start
                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
