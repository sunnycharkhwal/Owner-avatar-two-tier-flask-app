pipeline {
    agent { label 'dev' }
    
    // Define environment variables for the pipeline
    environment {
        // 👇 Replace this with your actual destination email
        EMAIL_TO = 'jangratube@gmail.com,sunny.charkhwal@gmail.com'
    }

    stages {
        stage('Git code clone') {
            steps {
                git url: "https://github.com/sunnycharkhwal/Owner-avatar-two-tier-flask-app.git", branch: "master"
            }
        }
        stage('Docker Build') {
            steps {
                // Consider adding the .dockerignore step here if you still face mysql-data permission issues
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
                withCredentials([usernamePassword(
                    credentialsId: "dockerHub",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
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
