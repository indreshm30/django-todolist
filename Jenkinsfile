pipeline {
    agent any

    environment {
        IMAGE_NAME = "django-todolist"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest'
            }
        }

        stage('List Docker Images') {
            steps {
                sh 'docker images | grep django-todolist'
            }
        }
    }

    post {
        failure {
            echo "❌ Pipeline failed! Please check logs above."
        }
        success {
            echo "✅ Pipeline completed successfully!"
        }
    }
}
