pipeline {
    agent any

    environment {
        APP_NAME = 'new-app-nti'
        REPO_URL = "https://github.com/Rewan0/jenkins-k8s-repo.git"
    }

    stages {
        stage('Getting Repo files') {
            steps {
                git branch: "main", credentialsId: 'dde', url: "${REPO_URL}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                        docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([
                    usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD'),
                    file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')
                ]) {
                    sh """
                        kubectl get nodes
                        sed 's|IMAGE_PLACEHOLDER|${DOCKER_USERNAME}/${APP_NAME}:${BUILD_NUMBER}|' k8s/deployment.yaml > deployment.yaml
                        kubectl apply -f deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        kubectl get pods
                        kubectl get services
                    """
                }
            }
        }
    }
}
