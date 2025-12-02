pipeline {
    agent any

    environment {
        SONAR_TOKEN   = 'sqa_11de5678255fedb92587190a210e00a696908983'
        SONAR_HOST_URL = 'http://localhost:9000'
        GIT_REPO      = 'https://github.com/SyedRehanAli25/sonarqube-demo.git'
    }

    stages {

        stage('Checkout') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build, Test & Coverage') {
            steps {
                sh "mvn clean verify"
            }
            post {
                always {
                    publishHTML(target: [
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JaCoCo Coverage',
                        keepAll: true
                    ])
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                timeout(time: 20, unit: 'MINUTES') {
                    sh """
                        mvn sonar:sonar \
                            -Dsonar.projectKey=my-app \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.token=${SONAR_TOKEN} \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.parser=JDT
                    """
                }
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
                                -Danalyzer.nvd.api.enabled=false \
                                -Danalyzer.nvd.forceUpdate=false \
                                -Danalyzer.nvd.apiKey=""
                                -DfailOnError=false
                        """
                    } catch (err) {
                        echo "OWASP scan failed (but allowed to continue): ${err}"
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'target/dependency-check-report.html', allowEmptyArchive: true
                }
            }
        }

    } // end stages

    post {
        always {
            junit 'target/surefire-reports/*.xml'
            echo 'Pipeline completed!'
        }
    }
}
