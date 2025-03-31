pipeline {
    agent {
        label 'worker-01'
    }

    parameters {
        string defaultValue: 'master', description: 'This is the branch to checkout the code', name: 'branch_name'
        choice choices: ['DEV', 'SIT', 'UAT'], description: 'Environment to be deployed', name: 'environment'
    }

    triggers {
        cron '00 20 * * *'
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder logRotator(numToKeepStr: '20')
        timestamps()
    }

    tools {
        jdk 'JAVA8'
        maven 'MAVEN3'
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${PATH}"
        
        // Tomcat Variables
        TOMCAT_SERVER = '54.174.217.216'
        TOMCAT_USER = 'ubuntu'
        TOMCAT_DEPLOY_PATH = '/opt/tomcat9/webapps'
        
        // Next.js Variables
        NEXT_SERVER = '54.174.217.216'
        NEXT_USER = 'ubuntu'
        NEXT_DEPLOY_PATH = '/var/www/nextjs-app'
        
        // Nexus Variables
        NEXUS_URL = 'http://54.174.217.216:3000'
        NEXUS_REPO = 'petclinic'
        NEXUS_CREDENTIALS = 'admin:password'
        
        // SonarQube
        SCANNER_HOME = tool 'sonarqube-scanner'
        
        // Email Notifications
        EMAIL_RECIPIENTS = 'sarita@techspira.co.in'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "master", credentialsId: 'github-token', url: 'https://github.com/saritaharihar/PetClinic.git'
            }
        }

        stage('Tests & Scans') {
            parallel {
                stage('Unit Testing') {
                    steps {
                        sh "mvn test"
                    }
                }
                stage('SonarQube Testing') {
                    steps {
                        withSonarQubeEnv(installationName: 'sonarqube-server') {
                            sh "$SCANNER_HOME/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                        }
                    }
                }
            }
        }
        
        stage('Package Code') {
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }
        
        stage('Upload Artifacts to Nexus') {
            steps {
                sh '''
                curl -u $NEXUS_CREDENTIALS -X POST "$NEXUS_URL/service/rest/v1/components?repository=$NEXUS_REPO" \
                -H "accept: application/json" -H "Content-Type: multipart/form-data" \
                -F "maven2.groupId=org.springframework.samples" \
                -F "maven2.artifactId=petclinic" \
                -F "maven2.version=${BUILD_ID}.0.0" \
                -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" \
                -F "maven2.asset1.extension=war"
                '''
            }
        }
        
        stage('Deploy to Tomcat') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                        chmod 600 "$SSH_KEY"
                        scp -i "$SSH_KEY" -o StrictHostKeyChecking=no target/petclinic.war $TOMCAT_USER@$TOMCAT_SERVER:$TOMCAT_DEPLOY_PATH/petclinic.war
                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $TOMCAT_USER@$TOMCAT_SERVER 'sudo systemctl restart tomcat'
                        '''
                    }
                }
            }
        }
        
        stage('Deploy Next.js to EC2') {
            steps {
                script {
                    withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                        sh '''
                        chmod 600 "$SSH_KEY"
                        scp -i "$SSH_KEY" -r * $NEXT_USER@$NEXT_SERVER:$NEXT_DEPLOY_PATH
                        ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $NEXT_USER@$NEXT_SERVER << 'EOF'
                            cd $NEXT_DEPLOY_PATH
                            npm install
                            npm run build
                            pm2 restart next-app || pm2 start npm --name "next-app" -- start
                            pm2 save
                        EOF
                        '''
                    }
                }
            }
        }
        
        stage('Run Ansible Deployment') {
            steps {
                ansiblePlaybook installation: 'ANSIBLE29', playbook: '/opt/ansible/deploy.yaml'
            }
        }
    }

    post {
        success {
            emailext subject: "✅ Deployment Successful",
                     body: "Jenkins successfully deployed both Java & Next.js applications.",
                     to: "$EMAIL_RECIPIENTS",
                     replyTo: 'no-reply@techspira.co.in',
                     from: 'jenkins@techspira.co.in',
                     credentialsId: 'Email-techspira'
        }
        failure {
            emailext subject: "❌ Deployment Failed",
                     body: "Jenkins failed to deploy one or both applications. Check logs for errors.",
                     to: "$EMAIL_RECIPIENTS",
                     replyTo: 'no-reply@techspira.co.in',
                     from: 'jenkins@techspira.co.in',
                     credentialsId: 'Email-techspira'
        }
    }
}
