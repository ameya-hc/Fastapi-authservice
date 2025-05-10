pipeline {
    agent any

    // Trigger pipeline on push to development branch
    triggers {
        pollSCM('H/5 * * * *')
    }

    // Environment variables
    environment {
        // Update these values with your specific details
        DOCKER_IMAGE = 'ameya133/fastapi-auth-service'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = 'dockerhub-creds' // Update this with your Jenkins credentials ID
        GITHUB_REPO = 'ameya-hc/Fastapi-authservice'
        BRANCH_NAME = 'devlopment'
        GITHUB_CREDENTIALS_ID = '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'  // Add your GitHub credentials ID here
    }

    stages {
        // Stage 1: Checkout code
        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}",
                    url: "https://github.com/${GITHUB_REPO}.git",
                    credentialsId: "${GITHUB_CREDENTIALS_ID}"
            }
        }

        // Stage 2: Build Docker image
        stage('Build') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        // Stage 3: Push to DockerHub
        stage('Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }
    }

    // Post-build actions
    post {
        success {
            echo "✅ Pipeline completed successfully!"
            echo "Docker image pushed: ${DOCKER_IMAGE}:${DOCKER_TAG}"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        always {
            // Clean up Docker images
            sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_IMAGE}:latest || true"
        }
    }
}
