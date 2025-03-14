pipeline {
    agent none  // Define agents per stage

    stages {
        stage('Build') {
            agent { label 'build-node' }
            steps {
                sh 'mvn clean package'  // Build the WAR file
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true  // Store WAR file
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'deploy-node' }
            steps {
                copyArtifacts projectName: 'test', filter: 'target/*.war', flatten: true  // Fetch WAR file

                sh '''
                cp *.war /home/ec2-user/apache-tomcat-9.0.102/webapps/
                /home/ec2-user/apache-tomcat-9.0.102/bin/shutdown.sh
                sleep 5
                /home/ec2-user/apache-tomcat-9.0.102/bin/startup.sh
                '''
            }
        }
    }
}