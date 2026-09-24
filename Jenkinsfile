// pipeline {

//     agent any

//     environment {

//         // Docker Hub image
//         DOCKER_IMAGE = "lyhov168/nextjs-test"

//         // Jenkins credential ID for Docker Hub
//         DOCKER_CREDENTIALS = "dockerhub-credentials"

//         // Jenkins credential ID for GitHub GitOps repository
//         GITHUB_CREDENTIALS = "github-gitops"

//         // GitOps repository
//         GITOPS_REPO = "https://github.com/joinlyhov-sketch/nextjs-gitops.git"

//         // GitOps Helm values file
//         GITOPS_VALUES_FILE = "charts/nextjs/values.yaml"

//         // Immutable Docker image tag
//         IMAGE_TAG = "build-${BUILD_NUMBER}"
//     }

//     stages {

//         stage('Checkout Application') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Install Dependencies') {
//             steps {
//                 sh 'npm ci'
//             }
//         }

//         stage('Lint') {
//             steps {
//                 sh 'npm run lint'
//             }
//         }

//         stage('Next.js Build') {
//             steps {
//                 sh 'npm run build'
//             }
//         }

//         stage('Docker Build') {
//             steps {
//                 sh """
//                     docker build \
//                         -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
//                         -t ${DOCKER_IMAGE}:latest \
//                         .
//                 """
//             }
//         }

//         stage('Docker Push') {
//             steps {

//                 withCredentials([
//                     usernamePassword(
//                         credentialsId: "${DOCKER_CREDENTIALS}",
//                         usernameVariable: 'DOCKER_USERNAME',
//                         passwordVariable: 'DOCKER_PASSWORD'
//                     )
//                 ]) {

//                     sh '''
//                         echo "$DOCKER_PASSWORD" | docker login \
//                             -u "$DOCKER_USERNAME" \
//                             --password-stdin

//                         docker push "$DOCKER_IMAGE:$IMAGE_TAG"

//                         docker push "$DOCKER_IMAGE:latest"

//                         docker logout
//                     '''
//                 }
//             }
//         }

//         stage('Update GitOps Repository') {
//             steps {

//                 dir('gitops') {

//                     echo "========================================"
//                     echo "Cloning GitOps repository"
//                     echo "========================================"

//                     git(
//                         url: "${GITOPS_REPO}",
//                         branch: 'master',
//                         credentialsId: "${GITHUB_CREDENTIALS}"
//                     )

//                     echo "========================================"
//                     echo "Updating image tag"
//                     echo "========================================"

//                     sh """
//                         sed -i 's/tag: .*/tag: "${IMAGE_TAG}"/' \
//                             "${GITOPS_VALUES_FILE}"
//                     """

//                     echo "Current image configuration:"

//                     sh """
//                         grep -A3 '^image:' "${GITOPS_VALUES_FILE}"
//                     """

//                     echo "========================================"
//                     echo "Configuring Git"
//                     echo "========================================"

//                     sh '''
//                         git config user.name "jenkins"
//                         git config user.email "jenkins@localhost"
//                     '''

//                     echo "========================================"
//                     echo "Committing GitOps change"
//                     echo "========================================"

//                     sh """
//                         git add "${GITOPS_VALUES_FILE}"

//                         git commit \
//                             -m "Update Next.js image to ${IMAGE_TAG}"
//                     """

//                     echo "========================================"
//                     echo "Pushing GitOps change"
//                     echo "========================================"

//                     sh '''
//                         git push origin master
//                     '''
//                 }
//             }
//         }
//     }

//     post {

//         success {
//             echo "========================================"
//             echo "CI/CD PIPELINE SUCCESS"
//             echo "========================================"
//             echo "Docker Image:"
//             echo "${DOCKER_IMAGE}:${IMAGE_TAG}"
//             echo ""
//             echo "GitOps:"
//             echo "Updated ${GITOPS_VALUES_FILE}"
//             echo ""
//             echo "Argo CD:"
//             echo "Will synchronize the GitOps change."
//             echo "========================================"
//         }

//         failure {
//             echo "========================================"
//             echo "PIPELINE FAILED"
//             echo "========================================"
//         }

//         always {
//             sh 'docker logout || true'
//         }
//     }
// }



pipeline {

    agent any

    // Declarative Pipeline normally performs an automatic SCM checkout.
    // We disable it because we explicitly checkout the application below.
    options {
        skipDefaultCheckout(true)
    }

    environment {

        // ============================================================
        // Docker Hub
        // ============================================================
        DOCKER_IMAGE = "lyhov168/nextjs-test"
        DOCKER_CREDENTIALS = "dockerhub-credentials"

        // ============================================================
        // GitHub GitOps Repository
        // ============================================================
        GITHUB_CREDENTIALS = "github-gitops"
        GITOPS_REPO = "https://github.com/joinlyhov-sketch/nextjs-gitops.git"
        GITOPS_VALUES_FILE = "charts/nextjs/values.yaml"

        // ============================================================
        // Immutable image tag
        // Example: build-18
        // ============================================================
        IMAGE_TAG = "build-${BUILD_NUMBER}"
    }

    stages {

        // ============================================================
        // 1. Checkout Application Repository
        // ============================================================
        stage('Checkout Application') {
            steps {
                echo "========================================"
                echo "Checking out Next.js application"
                echo "========================================"

                checkout scm
            }
        }

        // ============================================================
        // 2. Install Dependencies
        // ============================================================
        stage('Install Dependencies') {
            steps {
                echo "========================================"
                echo "Installing npm dependencies"
                echo "========================================"

                sh 'npm ci'
            }
        }

        // ============================================================
        // 3. Lint
        // ============================================================
        stage('Lint') {
            steps {
                echo "========================================"
                echo "Running ESLint"
                echo "========================================"

                sh 'npm run lint'
            }
        }

        // ============================================================
        // 4. Build Next.js
        // ============================================================
        stage('Next.js Build') {
            steps {
                echo "========================================"
                echo "Building Next.js application"
                echo "========================================"

                sh 'npm run build'
            }
        }

        // ============================================================
        // 5. Build Docker Image
        // ============================================================
        stage('Docker Build') {
            steps {
                echo "========================================"
                echo "Building Docker image"
                echo "========================================"

                sh """
                    docker build \
                        -t ${DOCKER_IMAGE}:${IMAGE_TAG} \
                        -t ${DOCKER_IMAGE}:latest \
                        .
                """
            }
        }

        // ============================================================
        // 6. Push Docker Image to Docker Hub
        // ============================================================
        stage('Docker Push') {
            steps {

                echo "========================================"
                echo "Logging into Docker Hub"
                echo "========================================"

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

                        echo "Pushing image:"
                        echo "$DOCKER_IMAGE:$IMAGE_TAG"

                        docker push "$DOCKER_IMAGE:$IMAGE_TAG"

                        echo "Pushing latest tag"

                        docker push "$DOCKER_IMAGE:latest"

                        docker logout
                    '''
                }
            }
        }

        // ============================================================
        // 7. Update GitOps Repository
        // ============================================================
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

                    echo "========================================"
                    echo "Current image configuration"
                    echo "========================================"

                    sh """
                        grep -A5 '^image:' "${GITOPS_VALUES_FILE}"
                    """

                    echo "========================================"
                    echo "Configuring Git"
                    echo "========================================"

                    sh '''
                        git config user.name "jenkins"
                        git config user.email "jenkins@localhost"
                    '''

                    echo "========================================"
                    echo "Checking Git changes"
                    echo "========================================"

                    sh '''
                        git status
                        git diff
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
                    echo "Pushing GitOps change to GitHub"
                    echo "========================================"

                    /*
                     * IMPORTANT:
                     *
                     * The Jenkins Git credential used by the `git(...)`
                     * checkout does NOT automatically authenticate a
                     * later `git push` executed by `sh`.
                     *
                     * Therefore we explicitly inject the GitHub
                     * username + PAT for this push.
                     */

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${GITHUB_CREDENTIALS}",
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {

                        sh '''
                            git config credential.helper \
                                '!f() { \
                                    echo username=$GITHUB_USERNAME; \
                                    echo password=$GITHUB_TOKEN; \
                                }; f'

                            git push origin master

                            git config --unset credential.helper || true
                        '''
                    }
                }
            }
        }
    }

    // ================================================================
    // POST ACTIONS
    // ================================================================
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
            echo "Will detect the GitOps repository change"
            echo "and synchronize the Kubernetes deployment."

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
