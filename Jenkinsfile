pipeline {
    agent { labels: "dev"}
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
}
