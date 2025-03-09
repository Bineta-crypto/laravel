pipeline {
    agent any

    environment {
        REGISTRY = 'docker.io'
        IMAGE_NAME = 'binta9619/laravel'
        IMAGE_TAG = 'latest'
        GIT_REPO = 'https://github.com/Bineta-crypto/laravel.git'
        DOCKER_CREDENTIALS = credentials('docker-credentials')
        KUBE_CONFIG = credentials('kubeconfig')
    }

    stages {
        stage('Récupération du code source') {
            steps {
                git branch: 'main', url: "${GIT_REPO}"
            }
        }

        stage('Installation des dépendances Laravel') {
            steps {
                sh 'composer install --no-interaction --prefer-dist --optimize-autoloader'
                sh 'php artisan key:generate'
            }
        }

        stage('Installation & build du frontend Node.js') {
            steps {
                sh 'npm ci'
                sh 'NODE_ENV=production npm run build'
            }
        }

        stage('Tests unitaires & IHM') {
            steps {
                sh 'php artisan test'
                sh 'npm test'
            }
        }

        stage('Connexion à Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                }
            }
        }

        stage('Création de l\'image Docker') {
            steps {
                sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Push de l\'image Docker') {
            steps {
                withDockerRegistry([credentialsId: 'docker-credentials', url: "https://${REGISTRY}"]) {
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('Déploiement sur Kubernetes') {
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
            echo ' Déploiement réussi !'
        }
        failure {
            echo ' Échec du déploiement !'
        }
    }
}
