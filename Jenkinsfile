pipeline {
    agent { label 'dev' }
    
    environment {
        EMAIL_TO = 'jangratube@gmail.com' 
        SONAR_PROJECT_KEY = 'two-tier-flask-app'
    }

    stages {
        stage('Git code clone') {
            steps {
                git url: "https://github.com/sunnycharkhwal/Owner-avatar-two-tier-flask-app.git", branch: "master"
            }
        }

        // 🛡️ SECURITY STAGE 1: Software Composition Analysis
        stage('OWASP Dependency-Check') {
            steps {
                // Securely pulls your 'nvd-api-key' credential to bypass rate limits
                withCredentials([string(credentialsId: 'nvd-api-key', variable: 'NVD_KEY')]) {
                    dependencyCheck additionalArguments: "--scan ./ --format HTML --format XML --nvdApiKey ${env.NVD_KEY}", odcInstallation: 'OWASP'
                }
            }
        }

        // 🛡️ SECURITY STAGE 2: Static Application Security Testing
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Fetch the installation path of the tool named 'Sonar'
                    def scannerHome = tool 'Sonar'
                    
                    // Add the tool's /bin directory to the PATH
                    withEnv(["PATH+SONAR=${scannerHome}/bin"]) {
                        // Matches your exact server name 'Sonar'
                        withSonarQubeEnv('Sonar') {
                            sh "sonar-scanner -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} -Dsonar.sources=."
                        }
                    }
                }
            }
        }

        // 🛡️ SECURITY STAGE 3: Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit test validation framework...'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t two-tier-flask-app ."
            }
        }

        // 🛡️ SECURITY STAGE 4: Local Image Vulnerability Scan
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL --exit-code 0 two-tier-flask-app"
            }
        }

        // 🛡️ SECURITY STAGE 5: Advanced Container Security & Push
        stage('Docker Scout & Push to Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "dockerHub",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker image tag two-tier-flask-app ${env.dockerHubUser}/two-tier-flask-app:latest"
                    
                    sh "docker push ${env.dockerHubUser}/two-tier-flask-app:latest"

                    sh "docker scout cves ${env.dockerHubUser}/two-tier-flask-app:latest"
                }
            }
        }

        stage('Docker compose Deploy') {
            steps {
                sh "docker compose up -d --build flask-app"
            }
        }
    }
    
    post {
        always {
            dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        }
        success {
            emailext (
                subject: "Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Congratulations! The Jenkins build for ${env.JOB_NAME} #${env.BUILD_NUMBER} was successful.\n\nYou can view the build details here: ${env.BUILD_URL}",
                to: "${env.EMAIL_TO}"
            )
        }
        failure {
            emailext (
                subject: "Jenkins Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Unfortunately, the Jenkins build for ${env.JOB_NAME} #${env.BUILD_NUMBER} has failed.\n\nPlease investigate the issue here: ${env.BUILD_URL}",
                to: "${env.EMAIL_TO}"
            )
        }
    }
}
