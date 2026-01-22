pipeline {
    agent any

    environment {
        IMAGE_NAME = "bhavyachowdary756/flask-shopeasy"
        DOCKER_CREDS = "docker-creds"
        GIT_CREDS = "github-creds"
        MANIFEST_FILE = "k8s/deployment.yaml"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    env.IMAGE_TAG = "${BUILD_NUMBER}"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: DOCKER_CREDS,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update K8s Manifest') {
            steps {
                sh """
                sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' ${MANIFEST_FILE}
                """
            }
        }

        stage('Commit & Push to Git') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: GIT_CREDS,
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh """
                    git config user.name "BhavyachowdaryA"
                    git config user.email "bhavya.angalakurthi@gmail.com"
                    git add ${MANIFEST_FILE}
                    git commit -m "Update image to ${IMAGE_NAME}:${IMAGE_TAG}"
                    git push https://${GIT_USER}:${GIT_PASS}@github.com/BhavyachowdaryA/flask-shopeasy.git HEAD:main
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI completed. Argo CD will deploy automatically."
        }
    }
}

