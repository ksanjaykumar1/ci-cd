pipeline {
    agent any

    environment {
        DOCKER_CREDS = credentials('dockerhub-creds')
        REMOTE_HOST = credentials('remote-host')
        REMOTE_SERVER_USR = credentials('remote-server-user')
    }

    stages {

        stage('Welcome Message') {
            steps {
                echo "Jenkins pipeline is starting on branch ${env.GIT_BRANCH}"
            }
        }
        stage('logs') {
            steps {
                sh 'ls -la'
            }
        }

        stage('Docker Login') {
            steps {
                sh '''
                    echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin
                '''
            }
        }

        stage('Build Image with Compose') {
            steps {
                dir('DemoApi') {
                    sh 'docker compose build'
                }
            }
        }

        stage('Push Image') {
            steps {
                dir('DemoApi') {
                    sh 'docker compose push'
                }
            }
        }
        stage('ReDeploy') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${REMOTE_SERVER_USR}@${REMOTE_HOST} << 'ENDSSH'
                        docker pull ksanjayk/demoapi:latest
                        docker service update --force --image ksanjayk/demoapi:latest demoapp_demoapi
                        ENDSSH       
                    """  
                }
                    
            }
        }
    }

    post {
        always {
            echo 'Logging out from Docker Hub...'
            sh 'docker logout'
        }
    }
}
