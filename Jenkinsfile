pipeline {
    agent any

    environment {
        DEPLOYMENT_NAME = "django-todolist"   // K8s Deployment name
        CONTAINER_NAME  = "django-todolist"   // Container name in deployment.yaml
        IMAGE_NAME      = "django-todolist:${BUILD_NUMBER}" // Docker image with build number
        KIND_CLUSTER    = "kind"              // Kind cluster name
        K8S_NAMESPACE   = "default"           // Namespace (change if needed)
    }

    stages {
        
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/indreshm30/django-todolist.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t $IMAGE_NAME .
                """
            }
        }

        stage('Load Image to Kind') {
            steps {
                sh """
                kind load docker-image $IMAGE_NAME --name $KIND_CLUSTER
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/$DEPLOYMENT_NAME $CONTAINER_NAME=$IMAGE_NAME -n $K8S_NAMESPACE
                kubectl rollout status deployment/$DEPLOYMENT_NAME -n $K8S_NAMESPACE
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                kubectl get pods -n $K8S_NAMESPACE
                kubectl get svc -n $K8S_NAMESPACE
                """
            }
        }

        stage('Integration Test') {
            steps {
                script {
                    // Get service IP (works for NodePort/LoadBalancer in Kind)
                    def service_url = sh(script: "kubectl get svc django-todolist-service -n $K8S_NAMESPACE -o jsonpath='{.spec.clusterIP}'", returnStdout: true).trim()
                    sh """
                    echo "Testing application response at http://$service_url"
                    curl --fail http://$service_url || (echo 'Integration test failed!' && exit 1)
                    """
                }
            }
        }
    }
}
