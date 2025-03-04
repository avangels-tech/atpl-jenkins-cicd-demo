pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            defaultContainer 'build-tools'
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/avangels-tech/atpl-jenkins-cicd-demo.git', branch: 'main'
            }
        }
        stage('Debug Credentials') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo "Username: $DOCKER_USERNAME"'
                    sh 'echo "Password (base64): $(echo $DOCKER_PASSWORD | base64)"'
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    container('docker') {
                        sh 'docker build -t avangelstech/k8s-jenkins-demo:${BUILD_NUMBER} .'
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push avangelstech/k8s-jenkins-demo:${BUILD_NUMBER}'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh 'sed -i "s/latest/${BUILD_NUMBER}/g" k8s-deployment.yaml'
                    sh 'kubectl apply -f k8s-deployment.yaml'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed!'
        }
    }
}
