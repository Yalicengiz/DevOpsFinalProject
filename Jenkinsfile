pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = 'docker-hub-credentials'
        KUBE_CONFIG = credentials('kube-config-file')
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("klenroixtia/my-project:${env.BUILD_NUMBER}", "-f Dockerfile .")
                }
            }
        }
        
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS) {
                        docker.image("klenroixtia/my-project:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def kubeConfig = readFile "${KUBE_CONFIG}"
                    withKubeConfig([credentialsId: 'kube-config-file', kubeconfigContent: kubeConfig]) {
                        bat 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
        
        stage('Run Docker Container') {
            steps {
                script {
                    docker.image("klenroixtia/my-project:${env.BUILD_NUMBER}").run("-p 3000:3000")
                }
            }
        }
    }
}
