pipeline {
    agent {
        label 'build-node'
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building the application'
                // Define build steps here
                sh '/usr/bin/mvn clean package'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests'
                // Define test steps here
                sh 'mvn test'
                stash (name: 'NumberGuessGame', includes: "target/*.war")
            }
        }
        stage('Deploy') {
            agent {
                label 'deploy-node'
            }
            steps {
                echo 'Deploying the application'
                unstash 'NumberGuessGame'
                sh "sudo rm -rf /home/ec2-user/apache-tomcat-9.0.102/webapps/*.war"
                sh "sudo mv target/NumberGuessGame-1.0-SNAPSHOT.war /home/ec2-user/apache-tomcat-9.0.102/webapps/"
                sh "sudo systemctl daemon-reload"
                sh "sudo /home/ec2-user/apache-tomcat-9.0.102/bin/shutdown.sh"
                sh "sudo /home/ec2-user/apache-tomcat-9.0.102/bin/startup.sh"
            }
        }
    }
    post {
        success {
            mail to: "timothytoweh1@gmail.com",
            subject: "Jenkins CI-CD Successful",
            body: "Jenkins CI-CD was successfully built, tested, and deployed"
        }
        failure {
            mail to: "timothytoweh1@gmail.com",
            subject: "Jenkins CI-CD Failed",
            body: "Jenkins CI-CD failed, please investigate"
        }
    }
}