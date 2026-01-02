pipeline {
    agent any

    environment {
        IMAGE_NAME  = "shivang2111/myapp"
        IMAGE_TAG   = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout App Repo') {
            steps {
                checkout scm
            }
        }

        stage('Build Application') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      docker login -u $DOCKER_USER -p $DOCKER_PASS
                      docker build -t $IMAGE_NAME:$IMAGE_TAG .
                      docker push $IMAGE_NAME:$IMAGE_TAG
                    """
                }
            }
        }

        stage('Update ArgoCD Deployment Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    script {
                        // Set deploy repo URL
                        def DEPLOY_REPO = "github.com/shivang123/myapp-deploy.git"

                        sh """
                            rm -rf myapp-deploy
                            git clone https://$GIT_USER:$GIT_PASS@${DEPLOY_REPO}
                            cd myapp-deploy

                            # Update image tag in deployment.yaml
                            sed -i "s|image: shivang2111/myapp:.*|image: shivang2111/myapp:${IMAGE_TAG}|g" k8s/deployment.yaml

                            git config user.name "Jenkins CI"
                            git config user.email "jenkins@ci.local"

                            git add k8s/deployment.yaml
                            git commit -m "Update myapp image to ${IMAGE_TAG}"
                            git push origin main
                        """
                    }
                }
            }
        }
    }
}

