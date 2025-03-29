pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  // Set Java 8 as the environment
        PATH = "${JAVA_HOME}/bin:${PATH}"
        TOMCAT_SERVER = '3.84.213.251'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'password'  // Use Jenkins credentials instead of hardcoding
        TOMCAT_DEPLOY_PATH = '/otp/tomcat/webapps'
        WAR_FILE = 'target/*.war'
        EMAIL_RECIPIENTS = 'sarita@techspira.co.in'
    }

    tools {
        maven 'MAVEN3'  // Use the Maven installation configured in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/saritaharihar/PetClinic.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
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
                    def warExists = fileExists("${WAR_FILE}")
                    if (warExists) {
                        sh "scp ${WAR_FILE} ${TOMCAT_USER}@${TOMCAT_SERVER}:${TOMCAT_DEPLOY_PATH}"
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
