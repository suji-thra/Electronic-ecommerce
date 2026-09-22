pipeline {

    agent any

    environment {
        IMAGE_NAME = "suji120/ecommerce-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f kubernetes/namespace.yml
                    kubectl apply -f kubernetes/deployment.yml
                    kubectl apply -f kubernetes/service.yml

                    kubectl set image deployment/ecommerce-deployment \
                    ecommerce=${IMAGE_NAME}:${IMAGE_TAG} \
                    -n ecommerce

                    kubectl rollout status deployment/ecommerce-deployment \
                    -n ecommerce
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get nodes
                    kubectl get pods -n ecommerce
                    kubectl get svc -n ecommerce
                '''
            }
        }
    }

    post {
        success {
            echo 'E-Commerce application deployed successfully.'
        }

        failure {
            echo 'Deployment failed.'
        }
    }
}
