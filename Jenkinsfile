pipeline {
    agent any
    environment {
        APP_PORT = '9090' // Set the environment variable
    }
    stages {
        stage('Setup') {
            steps {
                script {
                    globalJobName = env.JOB_NAME // Save job name in a global variable
                    echo "Job name: ${globalJobName}"
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package' // Build the project
            }
        }
        stage('Integration Test') {
            parallel {
                stage('Running Application') {
                    agent any
                    steps {
                        script {
                            timeout(time: 60, unit: 'SECONDS') { // Stop after 60 seconds
                                try {
                                    dir("target") {
                                        sh 'java -jar contact.war' // Run the application
                                    }
                                } catch (Exception e) {
                                    echo "Application run stopped after timeout."
                                    currentBuild.result = 'SUCCESS'
                                }
                            }
                        }
                    }
                }
                stage('Running Test') {
                    steps {
                        sleep 30 // Wait 30 seconds for the application to start
                        sh 'mvn -Dtest=RestIT test' // Run only RestIT integration test
                    }
                }
            }
        }
    }
}

