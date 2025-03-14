pipeline {
    agent none  // Define agents per stage

    stages {
        stage('Build') {
            agent { label 'build-node' }  // Run build on build-node
            steps {
                sh 'mvn clean package'  // Build the WAR file
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'tomcat-node' }  // Deploy directly on the Tomcat server
            steps {
                sh '''
                cp target/*.war /home/ec2-user/apache-tomcat-9.0.102/webapps/
                /home/ec2-user/apache-tomcat-9.0.102/bin/shutdown.sh
                sleep 5
                /home/ec2-user/apache-tomcat-9.0.102/bin/startup.sh
                '''
            }
        }
    }
}