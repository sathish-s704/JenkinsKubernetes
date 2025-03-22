pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "naveen014/docker-app:latest"  // Change this to your dockerhub username 1!#
        CONTAINER_NAME = "docker-running-app"
        REGISTRY_CREDENTIALS = "docker-sathish"  // Jenkins docker credentials ID 2!#
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-Naveen-Selvakumar', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {  //3!# git credentials ID
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/sathish-s704/JenkinsKubernetes.git", branch: 'main' //4!#git user-name repository ID
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-sathish', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'  //docker ID 5!#
                }
            }
        }

        stage('Push to Container Registry') {
            steps {
                sh 'docker push $DOCKER_IMAGE'
            }
        }

        stage('Stop & Remove Existing Container') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$CONTAINER_NAME)" ]; then
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                    fi
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 5001:5000 --name $CONTAINER_NAME $DOCKER_IMAGE'
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}
