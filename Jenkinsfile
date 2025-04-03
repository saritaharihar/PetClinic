pipeline {
    agent { label 'worker-01' }

    environment {
        GITHUB_CREDENTIALS_ID = 'github-token' // Use GitHub token for API limits
        SONARQUBE_SCANNER = 'sonarqube-scanner'
        NEXUS_URL = 'http://54.86.98.91:3000'
        NEXUS_REPO = 'petclinic'
        TOMCAT_SERVER = '52.23.169.3'
        TOMCAT_PATH = '/opt/tomcat9/webapps'
        NEXTJS_SERVER = '52.23.169.3'
        NEXTJS_PATH = '/var/www/nextjs-app'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/your-org/petclinic.git', branch: 'master'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build & Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh '''
                mvn deploy:deploy-file -DgroupId=com.example -DartifactId=petclinic \
                    -Dversion=1.0 -Dpackaging=war \
                    -Dfile=target/petclinic.war \
                    -DrepositoryId=nexus -Durl=${NEXUS_URL}/repository/${NEXUS_REPO}
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no target/petclinic.war ubuntu@${TOMCAT_SERVER}:${TOMCAT_PATH}/petclinic.war
                ssh -o StrictHostKeyChecking=no ubuntu@${TOMCAT_SERVER} "sudo systemctl restart tomcat9"
                '''
            }
        }

        stage('Deploy Next.js App') {
            steps {
                sh '''
                rsync -avz --delete ./nextjs-app ubuntu@${NEXTJS_SERVER}:${NEXTJS_PATH}
                ssh -o StrictHostKeyChecking=no ubuntu@${NEXTJS_SERVER} "cd ${NEXTJS_PATH} && pm2 restart nextjs-app || pm2 start npm --name nextjs-app -- run start"
                '''
            }
        }

        stage('Run Ansible Deployment') {
            steps {
                sh '''
                ansible-playbook -i inventory deploy.yml
                '''
            }
        }
    }

    post {
        success {
            mail to: 'sarita@techspira.co.in',
                 subject: 'Jenkins Build Success: PetClinic',
                 body: 'The latest PetClinic build & deployment was successful.'
        }
        failure {
            mail to: 'sarita@techspira.co.in',
                 subject: 'Jenkins Build Failed: PetClinic',
                 body: 'The latest PetClinic build & deployment failed. Please check Jenkins logs.'
        }
    }
}
