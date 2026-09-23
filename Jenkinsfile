pipeline {

    agent any

    environment {

        // Docker Hub image
        DOCKER_IMAGE = "lyhov168/nextjs-test"

        // Jenkins credential ID for Docker Hub
        DOCKER_CREDENTIALS = "dockerhub-credentials"

        // Jenkins credential ID for GitHub GitOps repository
        GITHUB_CREDENTIALS = "github-gitops"

        // GitOps repository
        GITOPS_REPO = "https://github.com/joinlyhov-sketch/nextjs-gitops.git"

        // GitOps Helm values file
        GITOPS_VALUES_FILE = "charts/nextjs/values.yaml"

        // Immutable Docker image tag
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Application') {
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
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
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

                        docker push "$DOCKER_IMAGE:$IMAGE_TAG"

                        docker push "$DOCKER_IMAGE:latest"

                        docker logout
                    '''
                }
            }
        }

        stage('Update GitOps Repository') {
            steps {

                dir('gitops') {

                    echo "========================================"
                    echo "Cloning GitOps repository"
                    echo "========================================"

                    git(
                        url: "${GITOPS_REPO}",
                        branch: 'master',
                        credentialsId: "${GITHUB_CREDENTIALS}"
                    )

                    echo "========================================"
                    echo "Updating image tag"
                    echo "========================================"

                    sh """
                        sed -i 's/tag: .*/tag: "${IMAGE_TAG}"/' \
                            "${GITOPS_VALUES_FILE}"
                    """

                    echo "Current image configuration:"

                    sh """
                        grep -A3 '^image:' "${GITOPS_VALUES_FILE}"
                    """

                    echo "========================================"
                    echo "Configuring Git"
                    echo "========================================"

                    sh '''
                        git config user.name "jenkins"
                        git config user.email "jenkins@localhost"
                    '''

                    echo "========================================"
                    echo "Committing GitOps change"
                    echo "========================================"

                    sh """
                        git add "${GITOPS_VALUES_FILE}"

                        git commit \
                            -m "Update Next.js image to ${IMAGE_TAG}"
                    """

                    echo "========================================"
                    echo "Pushing GitOps change"
                    echo "========================================"

                    sh '''
                        git push origin master
                    '''
                }
            }
        }
    }

    post {

        success {
            echo "========================================"
            echo "CI/CD PIPELINE SUCCESS"
            echo "========================================"
            echo "Docker Image:"
            echo "${DOCKER_IMAGE}:${IMAGE_TAG}"
            echo ""
            echo "GitOps:"
            echo "Updated ${GITOPS_VALUES_FILE}"
            echo ""
            echo "Argo CD:"
            echo "Will synchronize the GitOps change."
            echo "========================================"
        }

        failure {
            echo "========================================"
            echo "PIPELINE FAILED"
            echo "========================================"
        }

        always {
            sh 'docker logout || true'
        }
    }
}