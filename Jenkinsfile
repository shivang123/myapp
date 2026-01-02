pipeline {
    agent any

    environment {
        IMAGE_NAME  = "shivang2111/myapp"
        IMAGE_TAG   = "${BUILD_NUMBER}"
        DEPLOY_REPO = "git@github.com:shivang123/myapp-deploy.git"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-ssh',
                    url: 'git@github.com:shivang123/myapp.git'
            }
        }

        stage('Build Application') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    docker login -u $DOCKER_USER -p $DOCKER_PASS
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update ArgoCD Deployment Repo') {
            steps {
                sshagent(['github-ssh']) {
                    sh '''
                    rm -rf myapp-deploy
                    git clone git@github.com:shivang123/myapp-deploy.git
                    cd myapp-deploy

                    sed -i "s|image: .*|image: shivang2111/myapp:${IMAGE_TAG}|g" deployment.yaml

                    git config user.email "jenkins@ci.local"
                    git config user.name "jenkins"

                    git add deployment.yaml
                    git commit -m "Update image to ${IMAGE_TAG}"
                    git push
                    '''
                }
            }
        }
    }
}

