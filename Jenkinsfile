pipeline {
    agent {
        kubernetes {
            label 'jenkins-agent'
            defaultContainer 'build-tools'
        }
    }
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/avangels-tech/atpl-jenkins-cicd-demo.git', branch: 'main'
            }
        }
        stage('Build and Push Docker Image') {
            container('docker') {  // Run Docker commands in the docker container
                steps {
                    sh 'docker build -t avangelstech/k8s-jenkins-demo:${BUILD_NUMBER} .'
                    sh 'echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin'
                    sh 'docker push avangelstech/k8s-jenkins-demo:${BUILD_NUMBER}'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            container('kubectl') {  // Run kubectl commands in the kubectl container
                steps {
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
