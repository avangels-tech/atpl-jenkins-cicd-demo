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
                    sh 'whoami'  // Check running user
                    sh 'ls -ld /home/jenkins/agent/workspace'  // Check workspace dir permissions
                    sh 'ls -la /home/jenkins/agent/workspace/k8s-cicd-demo@tmp || echo "Tmp dir not found"'  // Check tmp dir
                    sh 'mkdir -p /home/jenkins/agent/workspace/k8s-cicd-demo@tmp/test && touch /home/jenkins/agent/workspace/k8s-cicd-demo@tmp/test/testfile || echo "Cannot write to tmp"'  // Test write
                    sh 'cp /var/jenkins_home/.kube/config ~/.kube/config || echo "Using cluster default config"'
                    sh 'ls -la'
                    sh 'cat k8s-deployment.yaml || echo "File not found"'
                    sh 'kubectl version --client'
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
