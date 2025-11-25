pipeline {
    agent any

    environment {
        // SonarQube token and host URL
        SONAR_TOKEN = 'sqa_1df9f405f4354d9ceacd11fa326afa5f2f5e1c3a' // replace with your actual token
        SONAR_HOST_URL = 'http://172.23.12.12:9000/' 
        GIT_REPO = 'https://github.com/SyedRehanAli25/sonarqube-demo.git'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "$GIT_REPO", branch: 'main'
            }
        }

        stage('Build, Test & Coverage') {
            steps {
                sh "mvn clean verify"
            }
            post {
                always {
                    script {
                        publishHTML(target: [
                            reportDir: 'target/site/jacoco',
                            reportFiles: 'index.html',
                            reportName: 'JaCoCo Coverage',
                            keepAll: true
                        ])
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Direct Maven SonarQube scan without using Sonar-scanner CLI
                sh """
                    mvn sonar:sonar \
                        -Dsonar.projectKey=my-app \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_TOKEN \
                        -Dsonar.java.binaries=target/classes
                """
            }
        }

        stage('OWASP Dependency Scan') {
            steps {
                script {
                    try {
                        sh """
                            mvn org.owasp:dependency-check-maven:check \
                                -Dformat=HTML \
                                -DoutputDirectory=target \
                                -DupdateOnly=false \
                                -DdataDirectory=~/.dependency-check
                        """
                    } catch (err) {
                        echo "OWASP Dependency Check failed, continuing pipeline: ${err}"
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/dependency-check-report.html', allowEmptyArchive: true
                }
            }
        }
    } // end of stages

    post {
        always {
            junit 'target/surefire-reports/*.xml'
            echo 'Pipeline completed!'
        }
    }
}
