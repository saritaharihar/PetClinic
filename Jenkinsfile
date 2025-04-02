pipeline {
    agent { label 'worker-01' }

    tools {
        jdk 'JAVA8'
        maven 'MAVEN3'
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-8-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:$PATH"

        // Tomcat Variables
        TOMCAT_SERVER = '34.201.104.10'
        TOMCAT_USER = 'ubuntu'
        TOMCAT_DEPLOY_PATH = '/opt/tomcat9/webapps'

        // Next.js Variables
        NEXT_SERVER = '34.201.104.10'
        NEXT_USER = 'ubuntu'
        NEXT_DEPLOY_PATH = '/var/www/nextjs-app'

        // Nexus Variables
        NEXUS_URL = 'http://34.201.104.10:9000'
        NEXUS_REPO = 'petclinic'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout scmGit(branches: [[name: params.branch_name]], userRemoteConfigs: [[url: 'https://github.com/saritaharihar/PetClinic.git', credentialsId: 'github-token']])
                }
            }
        }

        stage('Unit Testing') {
            steps {
                sh "mvn test"
            }
        }

        // Optional SonarQube Analysis
        /* stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=petclinic \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        } */

        stage('Package Code') {
            steps {
                sh "mvn package -Dmaven.test.skip=true"
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                    curl -u $NEXUS_USER:$NEXUS_PASS -X POST "$NEXUS_URL/service/rest/v1/components?repository=$NEXUS_REPO" \
                    -H "accept: application/json" -H "Content-Type: multipart/form-data" \
                    -F "maven2.groupId=org.springframework.samples" \
                    -F "maven2.artifactId=petclinic" \
                    -F "maven2.version=${BUILD_ID}.0.0" \
                    -F "maven2.asset1=@${WORKSPACE}/target/petclinic.war" \
                    -F "maven2.asset1.extension=war"
                    """
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                    chmod 600 "$SSH_KEY"
                    scp -i "$SSH_KEY" -o StrictHostKeyChecking=no target/petclinic.war $TOMCAT_USER@$TOMCAT_SERVER:$TOMCAT_DEPLOY_PATH/petclinic.war
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $TOMCAT_USER@$TOMCAT_SERVER 'sudo systemctl restart tomcat'
                    """
                }
            }
        }

        stage('Deploy Next.js to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                    chmod 600 "$SSH_KEY"
                    scp -i "$SSH_KEY" -r .next package.json ecosystem.config.js $NEXT_USER@$NEXT_SERVER:$NEXT_DEPLOY_PATH
                    ssh -i "$SSH_KEY" -o StrictHostKeyChecking=no $NEXT_USER@$NEXT_SERVER << 'EOF'
                        cd $NEXT_DEPLOY_PATH
                        npm ci --omit=dev
                        npm run build
                        pm2 restart ecosystem.config.js || pm2 start ecosystem.config.js
                        pm2 save
                    EOF
                    """
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
            withCredentials([string(credentialsId: 'Email-techspira', variable: 'EMAIL_RECIPIENTS')]) {
                emailext subject: "✅ Deployment Successful",
                         body: "Jenkins successfully deployed both Java & Next.js applications.",
                         to: EMAIL_RECIPIENTS
            }
        }
        failure {
            withCredentials([string(credentialsId: 'Email-techspira', variable: 'EMAIL_RECIPIENTS')]) {
                emailext subject: "❌ Deployment Failed",
                         body: "Jenkins failed to deploy one or both applications. Check logs for errors.",
                         to: EMAIL_RECIPIENTS
            }
        }
    }
}
