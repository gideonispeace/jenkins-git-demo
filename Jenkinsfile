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
    }
}
