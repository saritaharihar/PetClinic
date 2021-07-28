pipeline {

agent {
  label 'master'
}

parameters {
  string description: 'Provide the branch name here', name: 'BRANCH_NAME'
  choice choices: ['DEV', 'SIT', 'UAT'], description: 'environment name', name: 'environment'
}

environment {
  DATA_VALUE = "/opt/sample/data"
}

options {
  buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
  disableConcurrentBuilds()
  timestamps()
  timeout(10)
}

triggers {
  cron '00 * * * *'
}

tools {
  jdk 'JAVA8'
  maven 'MAVEN3'
}

stages {
  stage('Code Checkout') {
    steps {
      git branch: '$BRANCH_NAME', credentialsId: 'github-credentials', url: 'https://github.com/gopishank/PetClinic.git'
    }
  }

  stage('Unit Testing') {
    steps {
      sh "mvn test"
    }
  }
  
  stage('SonarQube Scan'){
      environment {
          SCANNER_HOME = tool 'sonarqube-scanner'
      }
      steps {
          withSonarQubeEnv(installationName: 'sonarqube-server'){
              sh "$SCANNER_HOME/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
          }
      }
  }
  
  stage('Package Code'){
      steps {
          sh "mvn package -Dmaven.test-skip=true"
      }
  }
  
  stage('Upload Artifacts'){
      steps {
          sh '''
          curl -u admin:admin123 POST "http://ec2-52-21-37-186.compute-1.amazonaws.com:8081/service/rest/v1/components?repository=petclinic" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "maven2.groupId=org.springframework.samples" -F "maven2.artifactId=petclinic" -F "maven2.version=${BUILD_ID}.0.0" -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" -F "maven2.asset1.extension=war"
          '''
      }
  }
  
  stage('Ansible Deploy'){
      steps {
          ansiblePlaybook installation: 'ANSIBLE29', playbook: '/opt/ansible/deploy.yaml'
      }
  }
  
}


post {
  always {
    echo "This block always runs"
  }
  success {
    echo "This block only runs when the jenkins pipeline is successful"
  }
  failure {
    echo "This block only runs when the jenkins pipeline has failed"
  }
}

}
