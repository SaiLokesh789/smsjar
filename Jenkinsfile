pipeline {
    agent any

    environment {
        APP_NAME = "sms"
        APP_NAMESPACE = "sms-namespace"
        IMAGE_NAME = "smsimg"
        IMAGE_TAG = "${BUILD_NUMBER}${BUILD_NUMBER}"
        APP_PORT = 8100
        NODE_PORT = 30080
        REPLICA_COUNT = 2
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

                        sh "envsubst < k8s/namespace-template.yaml > namespace.yaml"
                        sh "envsubst < k8s/deployment-template.yaml > deployment.yaml"
                        sh "envsubst < k8s/service-template.yaml > service.yaml"

                        sh "kubectl apply -f namespace.yaml --validate=false"
                        sh "kubectl apply -f deployment.yaml --validate=false"
                        sh "kubectl apply -f service.yaml --validate=false"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✔ Completed: Checkout → Build → Deploy"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
