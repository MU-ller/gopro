pipeline {
    agent any
    
    environment {
        GO_VERSION = '1.19'
        APP_NAME = 'my-go-web-app'
        DOCKER_REGISTRY = 'your-registry'  // Optional: if using Docker
        K8S_NAMESPACE = 'my-go-web-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup Go') {
            steps {
                sh '''
                    go version
                    go mod download
                '''
            }
        }
        
        stage('Build') {
            steps {
                sh 'go build -o ${APP_NAME} .'
            }
        }
        
        stage('Test') {
            steps {
                sh 'go test ./... -v'
            }
        }
        
        stage('Build Docker Image') {
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            when {
                expression { env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'master' }
            }
            steps {
                sh """
                    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER} -n ${K8S_NAMESPACE}
                    kubectl rollout status deployment/${APP_NAME} -n ${K8S_NAMESPACE}
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()  // Clean workspace
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
