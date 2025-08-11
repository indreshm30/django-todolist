pipeline {
    agent any

    environment {
        APP_NAME = "django-todolist"
        BUILD_TAGGED_IMAGE = "${APP_NAME}:${BUILD_NUMBER}"
        KIND_CLUSTER_NAME = "django-cluster"
        DEPLOYMENT_NAME = "django-todolist-deployment"
        NAMESPACE = "default"
        SERVICE_NAME = "django-todolist-service"
    }

    stages {
        
        stage('Checkout - Pull latest code') {
            steps {
                git branch: 'main', url: 'https://github.com/indreshm30/django-todolist.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${BUILD_TAGGED_IMAGE} .
                """
            }
        }

        stage('Load Image to Kind') {
            steps {
                sh """
                kind load docker-image ${BUILD_TAGGED_IMAGE} --name ${KIND_CLUSTER_NAME}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                # Update deployment with new image
                kubectl set image deployment/${DEPLOYMENT_NAME} ${APP_NAME}=${BUILD_TAGGED_IMAGE} -n ${NAMESPACE} || true
                kubectl apply -f k8s/ -n ${NAMESPACE}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                echo "Checking pods..."
                kubectl get pods -n ${NAMESPACE}
                
                echo "Waiting for rollout to finish..."
                kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=60s
                
                echo "Services:"
                kubectl get svc -n ${NAMESPACE}
                """
            }
        }

        stage('Integration Test') {
            steps {
                script {
                    def serviceUrl = sh(
                        script: "kubectl get svc ${SERVICE_NAME} -n ${NAMESPACE} -o jsonpath='{.spec.clusterIP}:{.spec.ports[0].port}'",
                        returnStdout: true
                    ).trim()
                    
                    echo "Testing application at ${serviceUrl}..."
                    sh """
                    curl --fail http://${serviceUrl} || (echo 'Integration test failed!' && exit 1)
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
