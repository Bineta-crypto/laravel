pipeline {
    agent any

    environment {
        REGISTRY = 'mon-registry.com'
        IMAGE_NAME = 'gestion-etablissement'
        IMAGE_TAG = 'latest'
        KUBE_CONFIG = credentials('kubeconfig') // Stocké dans Jenkins
        NAMESPACE = 'default'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:monrepo.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest' // Ajoute tes tests ici
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-credentials', url: "https://${REGISTRY}"]) {
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh """
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Déploiement réussi sur Kubernetes !"
        }
        failure {
            echo "Échec du pipeline."
        }
    }
}
