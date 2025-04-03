pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/saritaharihar/PetClinic.git'
        CREDENTIALS_ID = 'github-token'  // Ensure this matches your stored credentials
    }

    stages {
        stage('Checkout') {
            steps {
                git url: GIT_REPO, credentialsId: CREDENTIALS_ID
            }
        }
        
        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
