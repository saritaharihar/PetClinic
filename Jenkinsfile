pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  
        PATH = "${JAVA_HOME}/bin:${PATH}"
        
        // Tomcat Variables
        TOMCAT_SERVER = '54.158.1.245'  
        TOMCAT_USER = 'ubuntu'
        TOMCAT_DEPLOY_PATH = '/opt/tomcat9/webapps'
        
        // Next.js Variables
        NEXT_SERVER = '54.158.1.245'  // Replace with your EC2 instance IP
        NEXT_USER = 'ubuntu'
        NEXT_DEPLOY_PATH = '/var/www/nextjs-app'
        
        EMAIL_RECIPIENTS = 'sarita@techspira.co.in'
    }

    tools {
        maven 'MAVEN3'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/saritaharihar/PetClinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    def warFile = 'target/petclinic.war'

                    if (fileExists(warFile)) {
                        echo "Deploying WAR file: ${warFile}"

                        withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                            sh """
                                chmod 600 "${env.SSH_KEY}"
                                ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'mkdir -p ${TOMCAT_DEPLOY_PATH}'
                                scp -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${warFile} ${TOMCAT_USER}@${TOMCAT_SERVER}:${TOMCAT_DEPLOY_PATH}/petclinic.war
                                ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'sudo systemctl restart tomcat'
                                sleep 10
                                ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'curl -Is http://localhost:8080/petclinic | head -n 1'
                            """
                        }
                    } else {
                        error "WAR file not found!"
                    }
                }
            }
        }

        stage('Checkout Next.js Code') {
            steps {
                git branch: 'master', url: 'https://github.com/saritaharihar/PetClinic.git'  // Update your repo
            }
        }

        stage('Build Next.js App') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy Next.js to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'nextjs-ssh-key', keyFileVariable: 'NEXT_SSH_KEY')]) {
                        sh """
                            chmod 600 "${env.NEXT_SSH_KEY}"
                            
                            # Copy files to EC2
                            scp -i "${env.NEXT_SSH_KEY}" -r * ${NEXT_USER}@${NEXT_SERVER}:${NEXT_DEPLOY_PATH}

                            # SSH into EC2 and restart Next.js
                            ssh -i "${env.NEXT_SSH_KEY}" -o StrictHostKeyChecking=no ${NEXT_USER}@${NEXT_SERVER} << 'EOF'
                                cd ${NEXT_DEPLOY_PATH}
                                npm install
                                npm run build
                                pm2 restart next-app || pm2 start npm --name "next-app" -- start
                                pm2 save
                            EOF
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            emailext subject: "✅ Deployment Successful",
                     body: "Jenkins successfully deployed both Java & Next.js applications.",
                     to: "${EMAIL_RECIPIENTS}"
        }
        failure {
            emailext subject: "❌ Deployment Failed",
                     body: "Jenkins failed to deploy one or both applications. Check logs for errors.",
                     to: "${EMAIL_RECIPIENTS}"
        }
    }
}
