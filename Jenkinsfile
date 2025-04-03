pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your-repo/PetClinic.git'
            }
        }
        stage('Build with Maven') {
            steps {
                dir('PetClinic') {
                    sh 'mvn clean install'
                }
            }
        }
        stage('Run Tests') {
            steps {
                dir('PetClinic') {
                    sh 'mvn test'
                }
            }
        }
        stage('Package') {
            steps {
                dir('PetClinic') {
                    sh 'mvn package'
                }
            }
        }
    }
}
