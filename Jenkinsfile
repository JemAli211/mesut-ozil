pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }

    environment {
        IMAGE_NAME = "student-app"
        IMAGE_TAG  = "1.0"
        K8S_DIR    = "k8s"
    }

    stages {

       stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/JemAli211/mesut-ozil.git'
            }
        }


        stage('Build Maven') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Docker build (Minikube)') {
            steps {
                sh '''
                    echo ">>> Activation du moteur Docker de Minikube"
                    eval $(minikube docker-env)

                    echo ">>> Build de l'image Docker Spring Boot"
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    echo ">>> Vérification de l'image"
                    docker images | grep ${IMAGE_NAME}
                '''
            }
        }

        stage('Deployment Kubernetes') {
            steps {
                sh """
                    echo '>>> Déploiement MySQL'
                    kubectl apply -f ${K8S_DIR}/mysql-secret.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-pv-pvc.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-deployment.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-service.yaml

                    echo '>>> Déploiement Spring Boot'
                    kubectl apply -f ${K8S_DIR}/spring-deployment.yaml
                    kubectl apply -f ${K8S_DIR}/spring-service.yaml

                    echo '>>> Vérification des pods et services'
                    kubectl get pods
                    kubectl get svc
                """
            }
        }
    }

    post {
        success { echo "Pipeline exécuté avec succès 🎉" }
        failure { echo "❌ Erreur dans le pipeline" }
    }
}
