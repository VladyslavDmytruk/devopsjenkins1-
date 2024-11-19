pipeline {
    agent any
    environment {
        APP_PORT = '9090'
        JOB_NAME = "${env.JOB_NAME}"
        TARGET_DIR = "${env.WORKSPACE}/target" // Using WORKSPACE environment variable
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        timeout(time: 120, unit: 'SECONDS') { // Timeout to allow application to start
                            script {
                                try {
                                    // Launch the application from the target directory
                                    sh "java -jar ${env.TARGET_DIR}/contact.war --server.port=${APP_PORT} &"
                                    echo "Application started successfully from ${env.TARGET_DIR}."
                                } catch (Exception e) {
                                    echo "Application startup failed or timed out."
                                    error("Aborting parallel stages.")
                                }
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        // Wait for the application to start
                        script {
                            def retries = 10
                            def sleepInterval = 10
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
                                echo "Waiting for application to be ready... (Retry ${i + 1}/${retries})"
                                sleep sleepInterval
                            }

                            if (!isUp) {
                                error("Application is not reachable on port ${APP_PORT}. Failing the build.")
                            }
                        }
                        // Run tests from the target directory
                        sh "mvn -Dtest=RestIT test"
                    }
                }
            }
        }
    }
}
