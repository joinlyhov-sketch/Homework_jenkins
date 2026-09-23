pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "makarajr126/nextjs-app"
        DOCKER_CREDENTIALS = "dockerhub-credentials"
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

        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }

        stage('Next.js Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                """
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "CI SUCCESS"
            echo "Docker image: ${DOCKER_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo "CI FAILED"
        }

        always {
            sh 'docker logout || true'
        }
    }
}