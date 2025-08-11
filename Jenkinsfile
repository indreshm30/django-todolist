pipeline {
    agent any
    environment {
        IMAGE_NAME = "django-todolist"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CLUSTER_NAME = "django-cluster"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .
                    docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest
                    
                    echo "Docker images built:"
                    docker images | grep $IMAGE_NAME
                '''
            }
        }
        stage('Load Image to Kind') {
            steps {
                sh '''
                    echo "Loading image into kind cluster..."
                    kind load docker-image $IMAGE_NAME:$IMAGE_TAG --name $CLUSTER_NAME
                    kind load docker-image $IMAGE_NAME:latest --name $CLUSTER_NAME
                    
                    echo "Verifying image in cluster:"
                    docker exec -it $CLUSTER_NAME-control-plane crictl images | grep django-todolist
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying to Kubernetes..."
                    
                    # Update deployment with new image tag
                    sed -i "s|image: django-todolist:.*|image: django-todolist:${IMAGE_TAG}|g" k8s/deployment.yaml
                    
                    # Apply deployment
                    kubectl apply -f k8s/deployment.yaml
                    
                    # Wait for rollout to complete
                    kubectl rollout status deployment/django-todolist --timeout=300s
                    
                    echo "Deployment completed!"
                '''
            }
        }
        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "=== Deployment Status ==="
                    kubectl get deployments
                    kubectl get pods -l app=django-todolist
                    kubectl get services django-todolist-service
                    
                    echo "=== Application Health Check ==="
                    kubectl wait --for=condition=ready pod -l app=django-todolist --timeout=300s
                    
                    echo "✅ Application deployed and ready!"
                '''
            }
        }
        stage('Integration Test') {
            steps {
                sh '''
                    echo "=== Running Integration Tests ==="
                    
                    # Test application by port-forwarding in background
                    kubectl port-forward service/django-todolist-service 8082:80 &
                    FORWARD_PID=$!
                    
                    # Wait for port-forward to establish
                    sleep 5
                    
                    # Test the application
                    if curl -f http://localhost:8082 > /dev/null 2>&1; then
                        echo "✅ Application is responding correctly!"
                    else
                        echo "❌ Application health check failed"
                        kill $FORWARD_PID 2>/dev/null || true
                        exit 1
                    fi
                    
                    # Clean up port-forward
                    kill $FORWARD_PID 2>/dev/null || true
                    
                    echo "✅ Integration tests passed!"
                '''
            }
        }
    }
    post {
        success {
            echo '''
            🎉 ===============================================
            ✅ COMPLETE CI/CD PIPELINE SUCCESSFUL!
            ===============================================
            
            📋 Summary:
            - ✅ Source code checked out
            - ✅ Docker image built and tagged
            - ✅ Image loaded into Kubernetes cluster  
            - ✅ Application deployed to Kubernetes
            - ✅ Deployment verified and healthy
            - ✅ Integration tests passed
            
            🚀 Your Django TodoList is now running in Kubernetes!
            '''
        }
        failure {
            echo "❌ Pipeline failed! Check logs above for details."
            sh '''
                echo "=== Debugging Information ==="
                kubectl get pods -l app=django-todolist
                kubectl get events --sort-by='.lastTimestamp' | tail -10
            '''
        }
        always {
            sh '''
                echo "Cleaning up old Docker images..."
                docker image prune -f || echo "No images to prune"
            '''
        }
    }
}
