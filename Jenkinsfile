pipeline {
    agent none

    stages {
        stage('Clone Repository') {
            agent { label 'build-node' }
            steps {
                checkout([
                    $class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/timothy-toweh/NumberGuessGame']]
                ])
            }
        }

        stage('Build') {
            agent { label 'build-node' }
            steps {
                sh 'mvn clean package'
                archiveArtifacts artifacts: 'target/*.war', fingerprint: true
            }
        }

        stage('Test') {
            agent { label 'build-node' }
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            agent { label 'deploy-node' }
            steps {
                copyArtifacts(projectName: 'test', filter: 'target/*.war', flatten: true)
                sh '''
                TOMCAT_DIR=/opt/tomcat9/webapps
                WAR_FILE=NumberGuessGame.war
                
                if [ -f "$WAR_FILE" ]; then
                    cp $WAR_FILE $TOMCAT_DIR/
                    echo "Deployment successful!"
                else
                    echo "WAR file not found!"
                    exit 1
                fi
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details.'
        }
    }
}
