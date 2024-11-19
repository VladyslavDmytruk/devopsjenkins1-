pipeline {
    agent any
    environment {
        APP_PORT = '9090'
        JOB_NAME = "${env.JOB_NAME}"
        TARGET_DIR = "${env.WORKSPACE}/target"
    }
    stages {
        stage('Setup') {
            steps {
                sh 'sudo apt-get update && sudo apt-get install -y curl' // Adjust based on your OS
            }
        }
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
                        timeout(time: 120, unit: 'SECONDS') {
                            script {
                                try {
                                    sh "java -jar ${env.TARGET_DIR}/contact.war --server.port=${APP_PORT} &"
                                    echo "Application started successfully."
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
                                    echo "Application is reachable."
                                    break
                                }
                                echo "Retrying (${i + 1}/${retries})..."
                                sleep sleepInterval
                            }

                            if (!isUp) {
                                error("Application is not reachable.")
                            }
                        }
                        sh "mvn -Dtest=RestIT test"
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'pkill -f "java -jar"' // Clean up the application process
            echo "Application stopped."
        }
    }
}
