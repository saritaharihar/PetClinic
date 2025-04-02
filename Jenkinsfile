pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "34.201.104.10"
        APP_DIR = "/var/www/nextjs-app"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Linting & Tests') {
            steps {
                sh 'npm run lint'
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
                sh """
                scp -r .next package.json next.config.js ubuntu@${EC2_HOST}:${APP_DIR}
                ssh ${EC2_USER}@${EC2_HOST} << 'EOF'
                    cd ${APP_DIR}
                    npm install --omit=dev
                    pm2 restart nextjs-app || pm2 start "npm start" --name nextjs-app
                EOF
                """
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
