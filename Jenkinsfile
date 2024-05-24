pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'myapp:latest'
        DOCKER_REGISTRY = 'https://hub.docker.com/'
        DOCKER_CREDENTIALS_ID = 'your-docker-credentials-id'
        KUBECONFIG_CREDENTIALS_ID = 'your-kubeconfig-credentials-id'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([ credentialsId: "${DOCKER_CREDENTIALS_ID}", url: "${DOCKER_REGISTRY}" ]) {
                        sh "docker push ${DOCKER_IMAGE}"
                    }
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh 'npm test'
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                        sh 'kubectl apply -f deployment.yml'
                    }
                }
            }
        }
    }

    post {
        failure {
            stage('Rollback') {
                steps {
                    script {
                        // Assume you have a stable version tagged as 'previous-stable'
                        withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                            sh '''
                                kubectl set image deployment/myapp myapp=${DOCKER_REGISTRY}/myapp:previous-stable
                                kubectl rollout undo deployment/myapp
                            '''
                        }
                    }
                }
            }
        }
    }
}
