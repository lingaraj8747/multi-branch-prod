pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = "lingaraj8747/multibranch-flask-app"
        GIT_USER  = "lingaraj8747"
        GIT_EMAIL = "lingarajmaravalli6@gmail.com"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Image') {
            when { branch 'main' }
            steps {
                script {
                    env.IMAGE_TAG = "build-${BUILD_NUMBER}"
                    
                }

                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin <<< "$DOCKER_PASS"
                        docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update k8s Manifest') {
            when { branch 'main' }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'github-cred',
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                            set -e
                            git config user.name "${GIT_USER}"
                            git config user.email "${GIT_EMAIL}"
                            git fetch origin
                            git checkout main
                            git reset --hard origin/main
                            sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' k8s/deployment.yaml
                            git add k8s/deployment.yaml
                            git diff --cached --quiet || git commit -m "update image ${IMAGE_TAG}"
                            git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/lingaraj8747/k8s-multi-branch-prod.git main
                        """
                    }
                }
            }
        }
    }
}
