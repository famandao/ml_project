pipeline {
    agent any
    
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        REGISTRY = 'fama300'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/famandao/ml_project.git'
            }
        }
        
        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t ${REGISTRY}/backend:${BUILD_NUMBER} .'
                    sh 'docker tag ${REGISTRY}/backend:${BUILD_NUMBER} ${REGISTRY}/backend:latest'
                }
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t ${REGISTRY}/frontend:${BUILD_NUMBER} .'
                    sh 'docker tag ${REGISTRY}/frontend:${BUILD_NUMBER} ${REGISTRY}/frontend:latest'
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push ${REGISTRY}/backend:${BUILD_NUMBER}'
                    sh 'docker push ${REGISTRY}/backend:latest'
                    sh 'docker push ${REGISTRY}/frontend:${BUILD_NUMBER}'
                    sh 'docker push ${REGISTRY}/frontend:latest'
                }
            }
        }
        
        stage('Update Manifests') {
            steps {
                sh '''
                    sed -i "s|image:.*/backend.*|image: ${REGISTRY}/backend:${BUILD_NUMBER}|g" manifests/backend/deployment.yaml
                    sed -i "s|image:.*/frontend.*|image: ${REGISTRY}/frontend:${BUILD_NUMBER}|g" manifests/frontend/deployment.yaml
                    
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    git add manifests/
                    git commit -m "Update image tags to ${BUILD_NUMBER}"
                    git push origin main
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Build successful! Images pushed and manifests updated.'
        }
        failure {
            echo 'Build failed! Check logs.'
        }
    }
}