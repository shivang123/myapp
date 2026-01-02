pipeline {
    agent any

    environment {
        IMAGE_NAME = "shivang2111/myapp"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
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
                echo "Next step: update myapp-deploy repo"
            }
        }
    }
}

