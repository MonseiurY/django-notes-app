pipeline {
    // The "agent" section specifies where the pipeline should run (on any available agent in this case).
    agent any

    stages {
        stage("clone the code") {
            steps {
                echo "Cloning the code" 
                git url: "https://github.com/MonseiurY/django-notes-app.git", branch: "main"
            }
        }
        
        stage("build") {
            steps {
                echo "Building the image"
                sh "docker build -t notes-app ."
            }
        }
        
        stage("push image to docker hub") {
            steps {
                echo "Pushing the image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "docker tag notes-app ${env.dockerHubUser}/notes-app:latest"
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker push ${env.dockerHubUser}/notes-app:latest"
                }
            }
        }
        
        stage("deploy") {
            steps {
                echo "Deploying the image"
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
