stage('Deploy to Tomcat') {
    steps {
        script {
            def warFile = 'target/petclinic.war'

            if (fileExists(warFile)) {
                echo "Deploying WAR file: ${warFile}"

                withCredentials([sshUserPrivateKey(credentialsId: 'tomcat-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        chmod 600 "${env.SSH_KEY}"
                        ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} 'rm -rf ${TOMCAT_DEPLOY_PATH}/petclinic'
                        scp -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${warFile} ${TOMCAT_USER}@${TOMCAT_SERVER}:${TOMCAT_DEPLOY_PATH}/petclinic.war
                        ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} '
                            sudo chown -R tomcat:tomcat ${TOMCAT_DEPLOY_PATH}/petclinic.war;
                            sudo chmod -R 755 ${TOMCAT_DEPLOY_PATH}/petclinic.war;
                            sudo systemctl restart tomcat
                        '
                        sleep 10
                        ssh -i "${env.SSH_KEY}" -o StrictHostKeyChecking=no ${TOMCAT_USER}@${TOMCAT_SERVER} '
                            for i in {1..30}; do
                                if curl -Is http://localhost:8080/petclinic | grep "200 OK"; then
                                    echo "Application deployed successfully!";
                                    exit 0;
                                fi;
                                sleep 5;
                            done;
                            echo "Application failed to deploy!";
                            exit 1;
                        '
                    """
                }
            } else {
                error "WAR file not found!"
            }
        }
    }
}
