pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = "shivang2111"
        IMAGE_NAME = "myapp"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout App Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/shivang123/myapp.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }

        stage('Update Deployment YAML in myapp-deploy Repo') {
            steps {
                dir('myapp-deploy') {
                    git branch: 'main', url: 'https://github.com/shivang123/myapp-deploy.git'

                    sh """
                    sed -i 's|image:.*|image: $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG|' k8s/deployment.yaml
                    git config user.email "jenkins@mycompany.com"
                    git config user.name "Jenkins CI"
                    git add k8s/deployment.yaml
                    git commit -m "Update image to $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                    git push origin main
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

