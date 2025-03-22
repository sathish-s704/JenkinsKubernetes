pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = "sathish093/frontend-app:latest"  // Update with your DockerHub username
        BACKEND_IMAGE = "sathish093/backend-app:latest"
        FRONTEND_CONTAINER = "frontend-container"
        BACKEND_CONTAINER = "backend-container"
        REGISTRY_CREDENTIALS = "docker-sathish"  // Jenkins credentials ID for Docker login
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git_sathish-s704', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/sathish-s704/JenkinsKubernetes.git", branch: 'main'
                }
            }
        }

        stage('Build & Push Backend Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACKEND_IMAGE .'
                    withCredentials([usernamePassword(credentialsId: 'docker-sathish', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $BACKEND_IMAGE'
                    }
                }
            }
        }

        stage('Build & Push Frontend Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t $FRONTEND_IMAGE .'
                    withCredentials([usernamePassword(credentialsId: 'docker-sathish', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $FRONTEND_IMAGE'
                    }
                }
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$BACKEND_CONTAINER)" ]; then
                        docker stop $BACKEND_CONTAINER || true
                        docker rm $BACKEND_CONTAINER || true
                    fi

                    if [ "$(docker ps -aq -f name=$FRONTEND_CONTAINER)" ]; then
                        docker stop $FRONTEND_CONTAINER || true
                        docker rm $FRONTEND_CONTAINER || true
                    fi
                    '''
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh 'docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "Backend & Frontend deployment successful!"
        }
        failure {
            echo "Deployment failed. Check logs for details."
        }
    }
}

