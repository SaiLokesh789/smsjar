pipeline {
    agent any

    environment {
        APP_NAME        = "sms"
        APP_NAMESPACE   = "sms-namespace"
        DEPLOY_NAME     = "sms-deployment"
        SERVICE_NAME    = "sms-service"
        IMAGE_NAME      = "smsimg"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        APP_PORT        = 8100
        NODE_PORT       = 30080
        REPLICA_COUNT   = 2
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/SaiLokesh789/smsjar'
            }
        }

        stage('Cleanup Old Container (if exists)') {
            steps {
                sh '''
                    if [ "$(docker ps -aq -f name=${APP_NAME})" ]; then
                        echo "Container exists — stopping & removing..."
                        docker stop ${APP_NAME} || true
                        docker rm ${APP_NAME} || true
                    else
                        echo "No old container found."
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f Dockerfile ."
            }
        }

        stage('K8s Deployment') {
            steps {
                script {
                    withEnv(["KUBECONFIG=$HOME/.kube/config"]) {

                        echo "Updating YAML files dynamically..."

                        // Update Namespace file
                        sh """
                            sed -i '' 's/name: .*/name: ${APP_NAMESPACE}/' namespace.yaml
                        """

                        // Update Deployment file
                        sh """
                            sed -i '' 's/name: .*/name: ${DEPLOY_NAME}/' deployment.yaml
                            sed -i '' 's/namespace: .*/namespace: ${APP_NAMESPACE}/' deployment.yaml
                            sed -i '' 's/replicas: .*/replicas: ${REPLICA_COUNT}/' deployment.yaml
                            sed -i '' 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' deployment.yaml
                            sed -i '' 's/containerPort: .*/containerPort: ${APP_PORT}/' deployment.yaml
                        """

                        // Update Service file
                        sh """
                            sed -i '' 's/name: .*/name: ${SERVICE_NAME}/' service.yaml
                            sed -i '' 's/namespace: .*/namespace: ${APP_NAMESPACE}/' service.yaml
                            sed -i '' 's/port: .*/port: ${APP_PORT}/' service.yaml
                            sed -i '' 's/targetPort: .*/targetPort: ${APP_PORT}/' service.yaml
                            sed -i '' 's/nodePort: .*/nodePort: ${NODE_PORT}/' service.yaml
                        """

                        echo "Applying Kubernetes manifests..."

                        sh "kubectl apply -f namespace.yaml --validate=false"
                        sh "kubectl apply -f deployment.yaml --validate=false"
                        sh "kubectl apply -f service.yaml --validate=false"

                        sh "kubectl rollout status deployment/${DEPLOY_NAME} -n ${APP_NAMESPACE}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✔ Completed: Build ${IMAGE_NAME}:${IMAGE_TAG} and deployed to Kubernetes"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
