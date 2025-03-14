pipeline {
    agent none  // Define agents per stage

    environment {
        SERVER_USER = "ec2-user"  // Change if using a different user
        SERVER_IP = "172.31.25.6"  // Use private IP (or public if necessary)
        TOMCAT_HOME = "/home/ec2-user/apache-tomcat-9.0.102"
    }

    stages {
        stage('Build') {
            agent { label 'build-node' }  // Run on 'build-node'
            steps {
                sh 'mvn clean package'  // Adjust based on your project
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true  // Save the built WAR file
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'deploy-node' }  // Run on 'deploy-node'
            steps {
                sh '''
                scp -o StrictHostKeyChecking=no target/*.war $SERVER_USER@$SERVER_IP:$TOMCAT_HOME/webapps/
                ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "$TOMCAT_HOME/bin/shutdown.sh"
                ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "sleep 5"
                ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP "$TOMCAT_HOME/bin/startup.sh"
                '''
            }
        }
    }
}