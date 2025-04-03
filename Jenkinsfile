pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '52.23.169.3'  // Deployment EC2 instance
        DEPLOY_PATH = '/var/www/nextjs-app'
        GITHUB_REPO = 'saritaharihar/PetClinic'  // Update with your repo name
        REPO_DIR = 'PetClinic'  // Directory name after cloning
    }

    stages {
        stage('Clone Repository') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh 'git clone https://$GITHUB_TOKEN@github.com/${GITHUB_REPO}.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir(REPO_DIR) {
                    sh 'npm install'
                }
            }
        }

        stage('Run Linting & Tests') {
            steps {
                dir(REPO_DIR) {
                    sh 'npm test'
                }
            }
        }

        stage('Build Next.js App') {
            steps {
                dir(REPO_DIR) {
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                dir(REPO_DIR) {
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
}
