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
                                -Dnvd.api.enabled=false \
                                -Dnvd.api.modifiedUrl="" \
                                -Dnvd.api.baseUrl="" \
                                -DcveUrlModified="" \
                                -DcveUrlBase="" \
                                -Danalyzer.dependencycheck.failOnError=false \
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

                    // ✅ Added ONLY this block (no other pipeline changes)
                    publishHTML(target: [
                        reportDir: 'target',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'Dependency-Check Report',
                        keepAll: true
                    ])
                }
            }
        }

    } // end stages

    post {
        always {
            junit 'target/surefire-reports/*.xml'
            echo 'Pipeline completed!'

            emailext(
                to: 'rehan.ali9325@gmail.com',
                subject: "Jenkins Build: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${currentBuild.currentResult}",
                mimeType: 'text/html',
                body: """
                    <h2>Jenkins Build Report</h2>

                    <p><b>Job:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>Status:</b> ${currentBuild.currentResult}</p>

                    <h3>Generated Reports</h3>
                    <ul>
                        <li><a href="${env.BUILD_URL}htmlreports/JaCoCo_Coverage/">JaCoCo Coverage Report</a></li>
                        <li><a href="${env.BUILD_URL}htmlreports/Dependency-Check_Report/">OWASP Dependency Report</a></li>
                        <li><a href="${env.BUILD_URL}">Full Build Logs</a></li>
                    </ul>

                    <p>Regards,<br/>Jenkins CI</p>
                """,
                attachmentsPattern: "target/dependency-check-report.html"
            )
        }
    }
}
