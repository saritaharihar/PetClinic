pipeline {
    agent { label 'worker-01' }

    environment {
        GITHUB_CREDENTIALS_ID = 'github-token'
        SONARQUBE_SERVER = 'http://54.86.98.91:9000'
        NEXUS_URL = 'http://54.86.98.91:8081'
        NEXUS_REPO = 'petclinic'
        TOMCAT_SERVER = '52.23.169.3'
        TOMCAT_WEBAPPS = '/opt/tomcat9/webapps'
        NEXTJS_SERVER = '52.23.169.3'
        NEXTJS_DIR = '/var/www/nextjs-app'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git credentialsId: GITHUB_CREDENTIALS_ID, url: 'https://github.com/your-org/petclinic.git', branch: 'master'
                }
            }
        }

        stage('Unit Tests & SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=petclinic'
                    }
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Upload to Nexus') {
            steps {
                sh """
                    mvn deploy -DaltDeploymentRepository=nexus::default::${NEXUS_URL}/repository/${NEXUS_REPO}/
                """
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh """
                    scp target/*.war ubuntu@${TOMCAT_SERVER}:${TOMCAT_WEBAPPS}/petclinic.war
                    ssh ubuntu@${TOMCAT_SERVER} "sudo systemctl restart tomcat9"
                """
            }
        }

        stage('Deploy Next.js App') {
            steps {
                sh """
                    ssh ubuntu@${NEXTJS_SERVER} "cd ${NEXTJS_DIR} && git pull origin main && npm install && npm run build && pm2 restart all"
                """
            }
        }

        stage('Ansible Deployment') {
            steps {
                sh 'ansible-playbook -i inventory deploy.yml'
            }
        }
    }

    post {
        success {
            mail to: 'sarita@techspira.co.in',
                 subject: 'Jenkins Build Success: PetClinic',
                 body: 'The latest build was successful!'
        }
        failure {
            mail to: 'sarita@techspira.co.in',
                 subject: 'Jenkins Build Failed: PetClinic',
                 body: 'Check Jenkins logs for errors.'
        }
    }
}
