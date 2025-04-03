pipeline {
    agent any

    environment {
        TOMCAT_USER = 'tomcat'
        TOMCAT_HOST = '52.23.169.3'
        TOMCAT_PORT = '8080'
        WAR_FILE = 'target/petclinic.war'
        DEPLOY_PATH = '/opt/tomcat9/webapps'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/saritaharihar/PetClinic.git'
            }
        }

        stage('Build Java Application') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                script {
                    sh """
                        scp ${WAR_FILE} ${TOMCAT_USER}@${TOMCAT_HOST}:${DEPLOY_PATH}/petclinic.war
                        ssh ${TOMCAT_USER}@${TOMCAT_HOST} "sudo systemctl restart tomcat9"
                    """
                }
            }
        }
    }
}
