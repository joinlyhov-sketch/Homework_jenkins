pipeline {

    agent any

    environment {
        IMAGE = "docker.io/yourname/nextjs-shop"
        IMAGE_TAG = "${GIT_COMMIT}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
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
                sh """
                    docker build \
                      -t ${IMAGE}:${IMAGE_TAG} \
                      .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                    docker push ${IMAGE}:${IMAGE_TAG}
                """
            }
        }

        stage('Update GitOps') {
            steps {
                // clone GitOps repo
                // modify image tag
                // git commit
                // git push
            }
        }
    }
}