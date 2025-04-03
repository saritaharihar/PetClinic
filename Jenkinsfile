pipeline {
    agent any

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '52.23.169.3'  // Deployment EC2 instance
        DEPLOY_PATH = '/var/www/nextjs-app'
        GITHUB_REPO = 'saritaharihar/PetClinic'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir() // Ensures a clean workspace before cloning
            }
        }

        stage('Clone Repository') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh 'git clone https://$GITHUB_TOKEN@github.com/${GITHUB_REPO}.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('PetClinic') {
                    sh 'npm install'
                }
            }
        }

        stage('Run Linting & Tests') {
            steps {
                dir('PetClinic') {
                    sh 'npm test'
                }
            }
        }

        stage('Build Next.js App') {
            steps {
                dir('PetClinic') {
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    sh """
                        scp -r PetClinic/.next PetClinic/static PetClinic/package.json PetClinic/pm2.config.js ${EC2_USER}@${EC2_HOST}:${DEPLOY_PATH}
                    """

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
