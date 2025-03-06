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
                        sh 'whoami'  // Should resolve to prok8adm (UID 1000)
                        sh 'id'  // Should show uid=1000 gid=994
                        sh 'ls -ld /'
                        sh 'ls -ld /root'
                        sh 'ls -l /var/run/docker.sock'
                        sh 'docker ps || echo "Docker access failed"'
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
                    sh 'chmod -R 777 /home/jenkins/agent/workspace || echo "Permission fix failed"'
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
