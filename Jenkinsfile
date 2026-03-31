pipeline {
    agent any

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
                    sudo podman build -t my-custom-app .
                '''
            }
        }

        stage('Run Custom Container') {
            steps {
                sh '''
                    echo "Running custom container"
                    sudo podman run --rm my-custom-app
                '''
            }
        }
    }
}
