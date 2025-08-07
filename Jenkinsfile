pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Code checked out successfully'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            python3 --version
                            pip3 --version
                        '''
                    } else {
                        bat '''
                            python --version
                            pip --version
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
                            docker --version
                            docker build -t django-todolist:${BUILD_NUMBER} .
                            docker tag django-todolist:${BUILD_NUMBER} django-todolist:latest
                        '''
                    } else {
                        bat '''
                            docker --version
                            docker build -t django-todolist:%BUILD_NUMBER% .
                            docker tag django-todolist:%BUILD_NUMBER% django-todolist:latest
                        '''
                    }
                }
            }
        }
        
        stage('Test Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            echo "Docker images built:"
                            docker images | grep django-todolist
                        '''
                    } else {
                        bat '''
                            echo Docker images built:
                            docker images | findstr django-todolist
                        '''
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
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
