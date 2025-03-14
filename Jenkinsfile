pipeline {
    agent none

    environment {
        APP_NAME = "NumberGuessGame"
        GIT_REPO = "https://github.com/timothy-toweh/NumberGuessGame.git"
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
        DEPLOY_DIR = "/home/ec2-user/apache-tomcat-9.0.102/webapps/"
    }

    stages {
        stage('Checkout Code') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Checking out code from GitHub..."
                    checkout scm
                }
            }
        }

        stage('Build') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Building the Java web application using Maven..."
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Unit Tests') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Running unit tests..."
                    sh 'mvn test'
                }
            }
            post {
                always {
                    script {
                        def testResults = sh(script: "ls -1 target/surefire-reports/*.xml 2>/dev/null | wc -l", returnStdout: true).trim()
                        if (testResults.toInteger() > 0) {
                            junit '**/target/surefire-reports/*.xml'
                        } else {
                            echo "No test results found. Skipping JUnit report."
                        }
                    }
                }
            }
        }

        stage('Stash Artifact') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Stashing the built WAR file..."
                    stash name: 'war-file', includes: 'target/*.war'
                }
            }
        }

        stage('Deploy to Tomcat') {
            agent { label 'deploy-node' }
            steps {
                script {
                    echo "Fetching the WAR file from stash..."
                    unstash 'war-file'

                    echo "Deploying to Apache Tomcat on the deploy node..."
                    sh """
                        # Stop Tomcat using Catalina script
                        sudo /home/ec2-user/apache-tomcat-9.0.102/bin/shutdown.sh || true

                        # Wait for Tomcat to fully shut down
                        sleep 5

                        # Remove old deployment
                        sudo rm -rf ${DEPLOY_DIR}ROOT.war
                        sudo rm -rf ${DEPLOY_DIR}ROOT/

                        # Move the new WAR file
                        sudo mv target/*.war ${DEPLOY_DIR}ROOT.war

                        # Start Tomcat using Catalina script
                        sudo /home/ec2-user/apache-tomcat-9.0.102/bin/startup.sh
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}