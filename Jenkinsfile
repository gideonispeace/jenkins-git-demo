pipeline {
    agent any

    environment {
        IMAGE_NAME = 'gannex/my-custom-app'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('From Git') {
            steps {
                sh '''
                    echo "Pipeline running from Git repo"
                    echo "User:"
                    whoami
                    echo "Hostname:"
                    hostname
                    echo "Date:"
                    date
                    echo "Workspace:"
                    pwd
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    echo "Listing files"
                    ls -la
                    echo "Polling test successful"
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    echo "Running Docker container test"
                    sudo docker run hello-world
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    echo "Building custom image"

cat > Dockerfile <<EOF
FROM alpine:latest
CMD ["echo", "Hello from my custom Jenkins image"]
EOF

                    sudo podman build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Run Custom Container') {
            steps {
                sh '''
                    echo "Running custom container"
                    sudo podman run --rm ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "Logging into Docker Hub"
                        echo "$DOCKER_PASS" | sudo podman login docker.io -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    echo "Pushing image to Docker Hub"
                    sudo podman push ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }
}
