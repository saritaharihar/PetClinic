pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'  // Use Java 8
        PATH = "${JAVA_HOME}/bin:${PATH}"
        TOMCAT_SERVER = '3.84.213.251'
        TOMCAT_USER = 'admin'
        TOMCAT_PASS = 'password'  // Use Jenkins credentials instead of hardcoding
        TOMCAT_DEPLOY_PATH = '/otp/tomcat/webapps'
        EMAIL_RECIPIENTS = 'sarita@techspira.co.in'
    }

    tools {
        maven 'MAVEN3'  // Use configured Maven installation
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
                    def warFile = sh(script: "ls target/*.war | head -n 1", returnStdout: true).trim()
                    if (warFile) {
                        echo "Deploying WAR file: ${warFile}"
                        sh "scp ${warFile} ${TOMCAT_USER}@${TOMCAT_SERVER}:${TOMCAT_DEPLOY_PATH}"
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
