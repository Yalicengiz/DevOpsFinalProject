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
                    // YAML dosyalarını Kubernetes'e uygulama
                    def kubeConfig = readFile "${KUBE_CONFIG}"
                    withKubeConfig([credentialsId: 'kube-config-file', kubeconfigContent: kubeConfig]) {
                        bat 'kubectl apply -f service.yaml'
                        bat 'kubectl apply -f configmap.yaml'
                        bat 'kubectl apply -f secret.yaml'
                        bat 'kubectl apply -f deployment.yaml'
                        bat 'kubectl apply -f network-policy.yaml'
                    }
                }
            }
        }
    }
}
