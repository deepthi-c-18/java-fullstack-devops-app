pipeline {
    agent { label 'docker' }

    environment {
        // Replace 'your_dockerhub_username' with your actual Docker Hub username
        DOCKERHUB_USER = 'deepthic18'
        IMAGE_NAME = 'my-nginx-app'
        IMAGE_TAG = 'v1'
        LATEST_TAG = 'latest'
        DOCKER_CREDENTIALS_ID = 'docker-hub-creds' // ID of credentials stored in Jenkins
    }

    stages {
        stage('Pull Source Code') {
            steps {
                echo 'Pulling source code from GitHub...'
                // Using the requested repository or your own
                git branch: 'main', url: 'https://github.com/deepthi-c-18/java-fullstack-devops-app.git'
                
                // If you are using the index.html from your local folder, you would push that to your own GitHub repo 
                // and replace the URL above with your GitHub URL.
            }
        }

        stage('Build the Image') {
            steps {
                echo 'Building the Docker Image...'
                // Assuming Dockerfile is in the root of the workspace
                sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('Change Tag Name') {
            steps {
                echo 'Changing the tag name of Docker Image...'
                // Changing the tag to include Docker Hub username and 'latest' tag
                sh 'docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USER}/${IMAGE_NAME}:${LATEST_TAG}'
            }
        }

        stage('Docker Login') {
            steps {
                echo 'Logging into Docker Hub...'
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker Image to Docker Hub...'
                sh 'docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${LATEST_TAG}'
            }
        }

        stage('Run the Container') {
            steps {
                echo 'Running the Container...'
                script {
                    // Check if container exists, stop and remove if it does
                    def containerExists = sh(script: "docker ps -a -q -f name=${IMAGE_NAME}-container", returnStdout: true).trim()
                    
                    if (containerExists) {
                        echo "Container already present. Stopping and removing..."
                        sh "docker stop ${IMAGE_NAME}-container || true"
                        sh "docker rm ${IMAGE_NAME}-container || true"
                    } else {
                        echo "Container not present. Running directly..."
                    }
                    
                    // Run the new container
                    sh "docker run -d --name ${IMAGE_NAME}-container -p 8080:80 ${DOCKERHUB_USER}/${IMAGE_NAME}:${LATEST_TAG}"
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up workspace and docker credentials...'
            sh 'docker logout'
        }
    }
}
