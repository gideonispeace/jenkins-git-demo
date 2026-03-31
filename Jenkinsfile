pipeline {
    agent any

    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'test', 'prod'], description: 'Target environment tag')
    }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        IMAGE_NAME = 'gannex/my-custom-app'
        IMAGE_TAG = 'latest'
        BUILD_TAG_VERSION = "${BUILD_NUMBER}"
        ENV_TAG = "${params.DEPLOY_ENV}"
        CONTAINER_NAME = "my-app-${params.DEPLOY_ENV}"
    }

    stages {
        stage('Source Validation') {
            steps {
                sh '''
                    echo "Pipeline running from Git repo"
                    whoami
                    hostname
                    date
                    pwd
                '''
            }
        }

        stage('Workspace Verification') {
            steps {
                sh '''
                    echo "Listing files"
                    ls -la
                    echo "Polling test successful"
                '''
            }
        }

        stage('Container Runtime Test') {
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
                    sudo podman tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:${BUILD_TAG_VERSION}
                    sudo podman tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:${ENV_TAG}
                '''
            }
        }

        stage('Run Image Validation') {
            steps {
                sh '''
                    echo "Running custom container validation"
                    sudo podman run --rm ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Registry Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "Logging into Docker Hub"
                        echo "$DOCKER_PASS" | sudo podman login docker.io -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    echo "Pushing latest image"
                    sudo podman push ${IMAGE_NAME}:${IMAGE_TAG}

                    echo "Pushing versioned image"
                    sudo podman push ${IMAGE_NAME}:${BUILD_TAG_VERSION}

                    echo "Pushing environment image"
                    sudo podman push ${IMAGE_NAME}:${ENV_TAG}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying application for environment: ${ENV_TAG}"

                    sudo podman rm -f ${CONTAINER_NAME} || true

                    sudo podman run -d --name ${CONTAINER_NAME} ${IMAGE_NAME}:${ENV_TAG}

                    echo "Deployment complete"
                    sudo podman ps
                '''
            }
        }
    }

    post {
        success {
            sh '''
                echo "Pipeline completed successfully"
                echo "Deployed container: ${CONTAINER_NAME}"
            '''
        }

        failure {
            sh '''
                echo "Pipeline failed"
            '''
        }

        always {
            sh '''
                echo "Pipeline finished"
                sudo podman images | head -10 || true
            '''
        }
    }
}
