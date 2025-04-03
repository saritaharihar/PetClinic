pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '52.23.169.3'  // Deployment EC2 instance
        DEPLOY_PATH = '/var/www/nextjs-app'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/yourusername/nextjs-app.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Linting & Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Next.js App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    // Transfer build files to EC2 instance
                    sh """
                        scp -r .next static package.json pm2.config.js ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}
                    """
                    
                    // Connect to EC2 and restart the Next.js app
                    sh """
                        ssh ${EC2_USER}@${EC2_HOST} << EOF
                        cd ${DEPLOY_PATH}
                        npm install --omit=dev
                        pm2 restart pm2.config.js || pm2 start pm2.config.js
                        EOF
                    """
                }
            }
        }
    }
}
