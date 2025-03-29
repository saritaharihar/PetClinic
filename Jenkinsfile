pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  
        PATH = "${JAVA_HOME}/bin:${PATH}"
        TOMCAT_SERVER = '52.91.243.23'
        TOMCAT_USER = 'ubuntu'
        TOMCAT_DEPLOY_PATH = '/opt/tomcat/webapps'
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
                            sh(script: """
                                chmod 600 $SSH_KEY
                                ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'mkdir -p ${TOMCAT_DEPLOY_PATH}'
                                scp -i $SSH_KEY -o StrictHostKeyChecking=no ${warFile} ${TOMCAT_USER}@${TOMCAT_SERVER}:${TOMCAT_DEPLOY_PATH}/petclinic.war
                                ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'sudo systemctl restart tomcat'
                            """, sensitive: true)
                        }
                    } else {
                        error "WAR file not found!"
                    }
                }
            }
        }
    }

    post {
        success {
            emailext subject: "Deployment Successful",
                     body: "Jenkins successfully deployed the application to Tomcat.",
                     to: "${EMAIL_RECIPIENTS}"
        }
        failure {
            emailext subject: "Deployment Failed",
                     body: "Jenkins failed to deploy the application. Check logs for errors.",
                     to: "${EMAIL_RECIPIENTS}"
        }
    }
}
