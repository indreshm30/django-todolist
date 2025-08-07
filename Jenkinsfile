pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out successfully'
            }
        }
        
        stage('Check Environment') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            echo "Running on Unix/Linux"
                            python3 --version || echo "Python3 not found"
                            docker --version || echo "Docker not found"
                        '''
                    } else {
                        bat '''
                            echo Running on Windows
                            docker --version
                        '''
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            docker build -t django-todolist:${BUILD_NUMBER} .
                            docker tag django-todolist:${BUILD_NUMBER} django-todolist:latest
                        '''
                    } else {
                        bat '''
                            docker build -t django-todolist:%BUILD_NUMBER% .
                            docker tag django-todolist:%BUILD_NUMBER% django-todolist:latest
                        '''
                    }
                }
            }
        }
        
        stage('List Docker Images') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'docker images | grep django-todolist || echo "No images found"'
                    } else {
                        bat 'docker images | findstr django-todolist'
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed!'
        }
        success {
            echo 'Pipeline succeeded! Docker image built successfully.'
        }
        failure {
            echo 'Pipeline failed! Check the logs above.'
        }
    }
}
