```groovy
pipeline {

    agent any

    environment {
        IMAGE = 'lyhov168/nextjs-test'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Test') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push ${IMAGE}:${IMAGE_TAG}
                        docker logout
                    '''
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully!"
            echo "Docker image: ${IMAGE}:${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline failed!"
        }
    }
}
```
