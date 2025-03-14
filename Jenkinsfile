pipeline {
    agent none

    environment {
        APP_NAME = "NumberGuessGame"
        GIT_REPO = "https://github.com/timothy-toweh/NumberGuessGame.git"
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
        DEPLOY_DIR = "/opt/tomcat/webapps/"
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
            post {
                failure {
                    echo "Build failed. Check the logs!"
                    error "Stopping pipeline due to failed build!"
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
                    junit '**/target/surefire-reports/*.xml'
                }
                failure {
                    echo "Unit tests failed. Stopping deployment."
                    error "Stopping pipeline due to failed tests!"
                }
            }
        }

        stage('Code Quality Check') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Checking code quality using Maven Checkstyle..."
                    sh 'mvn checkstyle:check'
                }
            }
            post {
                failure {
                    echo "Code quality check failed. Review errors before deploying."
                    error "Stopping pipeline due to code quality issues!"
                }
            }
        }

        stage('Archive Artifacts') {
            agent { label 'build-node' }
            steps {
                script {
                    echo "Archiving the built artifact..."
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        stage('Deploy to Tomcat') {
    agent { label 'deploy-node' }
    steps {
        script {
            echo "Deploying to Apache Tomcat on the deploy node..."
            sh """
                # Stop Tomcat using Catalina script
                sudo /usr/share/tomcat/bin/shutdown.sh || true
                
                # Wait for Tomcat to fully shut down
                sleep 5
                
                # Remove old deployment
                sudo rm -rf ${DEPLOY_DIR}ROOT.war
                sudo rm -rf ${DEPLOY_DIR}ROOT/
                
                # Deploy new war file
                sudo cp target/*.war ${DEPLOY_DIR}ROOT.war
                
                # Start Tomcat
                sudo /usr/share/tomcat/bin/startup.sh
            """
        }
    }
}
                }
            }
        
    

    post {
        success {
            echo "Deployment successful!"
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build and Deployment Success',
                 body: "The latest build and deployment of ${APP_NAME} was successful!"
        }
        failure {
            echo "Deployment failed!"
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build and Deployment Failed',
                 body: "The build and deployment of ${APP_NAME} failed. Please check the logs."
        }
    }
