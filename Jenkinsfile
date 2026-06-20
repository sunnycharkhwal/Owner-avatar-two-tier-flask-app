@Library('Shared') _
pipeline {
    agent { label 'dev' }
    
   

    stages {
        stage('Git code clone') {
            steps {
                 script {
                   clone("https://github.com/sunnycharkhwal/Owner-avatar-two-tier-flask-app.git", "master")
                }
            }
        }

        // 🛡️ SECURITY STAGE 1: Software Composition Analysis (Bulletproofed)
        stage('OWASP Dependency-Check') {
            steps {
                owaspDependencyCheck('nvd-api-key')
            }
        }

        // 🛡️ SECURITY STAGE 2: Static Application Security Testing
        stage('SonarQube Analysis') {
            steps {
                sonarQubeAnalysis('two-tier-flask-app')
            }
        }

        // 🛡️ SECURITY STAGE 3: Quality Gate
        stage('Quality Gate') {
            steps {
                qualityGateCheck()
            }
        }

        stage('Docker Build') {
            steps {
                 docker_build("two-tier-flask-app")
            }
        }

        // 🛡️ SECURITY STAGE 4: Local Image Vulnerability Scan
        stage('Trivy Image Scan') {
            steps {
                trivy_fs()
            }
        }

        // 🛡️ SECURITY STAGE 5: Advanced Container Security & Push
        stage('Docker Scout & Push to Hub') {
            steps {
                docker_push("dockerHub", "two-tier-flask-app")
            }
        }

        stage('Docker compose Deploy') {
            steps {
                docker_compose()
            }
        }
    }
    
    post {
        always {
            sendEmailNotification('jangratube@gmail.com')
        }
    }
}
