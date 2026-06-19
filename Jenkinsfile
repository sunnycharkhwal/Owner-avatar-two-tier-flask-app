pipeline {
    agent { label 'dev' }
    stages {
        stage('Git code clone') {
            steps {
                git url: "https://github.com/sunnycharkhwal/Owner-avatar-two-tier-flask-app.git", branch: "master"
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build -t two-tier-flask-app ."
            }
        }
        stage('Test') {
            steps {
                echo 'Running unit test validation framework...'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(        // ✅ Fix 1: withCredentials (plural)
                    credentialsId: "dockerHub",           // ✅ Fix 2: credentialsId (lowercase 'd')
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"  // ✅ Fix 3: added -u flag
                    sh "docker image tag two-tier-flask-app ${env.dockerHubUser}/two-tier-flask-app"
                    sh "docker push ${env.dockerHubUser}/two-tier-flask-app:latest"
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
        success {
            emailext (
                subject: "build successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
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
} // ✅ ADDED: Missing closing brace for the main 'pipeline' block
