pipeline {
    agent any
    environment {
        APP_PORT = '9090'
        TARGET_DIR = "${env.WORKSPACE}/target"
        JOB_NAME = "${env.JOB_NAME}"
    }
    options {
        timeout(time: 5, unit: 'MINUTES') // Ensure the pipeline does not hang indefinitely
    }
    stages {
        stage('Build') {
            steps {
                echo "Building the application..."
                sh 'mvn clean package'
            }
        }
        stage('Integration Test') {
            parallel {
                stage('Start Application') {
                    steps {
                        script {
                            try {
                                echo "Starting the application..."
                                sh "nohup java -jar ${TARGET_DIR}/contact.war --server.port=${APP_PORT} > app.log 2>&1 &"
                                echo "Application started successfully."
                            } catch (Exception e) {
                                echo "Failed to start the application."
                                error("Aborting integration test stages.")
                            }
                        }
                    }
                }
                stage('Run Tests') {
                    steps {
                        script {
                            echo "Installing curl..."
                            // Install curl before using it (for Debian/Ubuntu-based systems)
                            sh 'apt-get update && apt-get install -y curl' // For Debian/Ubuntu-based systems
                            // For RedHat/CentOS-based systems, use the following:
                            // sh 'yum install -y curl'

                            echo "Waiting for the application to be ready..."
                            def retries = 10
                            def isUp = false

                            for (int i = 0; i < retries; i++) {
                                def response = sh(
                                    script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${APP_PORT}",
                                    returnStdout: true
                                ).trim()

                                if (response == '200') {
                                    isUp = true
                                    echo "Application is reachable on port ${APP_PORT}."
                                    break
                                }
                                echo "Retry ${i + 1}/${retries}... Application not ready yet."
                                sleep 10
                            }

                            if (!isUp) {
                                error("Application is not reachable on port ${APP_PORT}.")
                            }

                            echo "Running integration tests..."
                            sh 'mvn -Dtest=RestIT test'
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Cleaning up resources..."
            script {
                try {
                    sh "pkill -f 'java -jar' || echo 'No running application processes to kill.'"
                } catch (Exception e) {
                    echo "Error during cleanup: ${e.message}"
                }
            }
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Please check logs for more details."
        }
    }
}
