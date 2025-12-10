pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }

    environment {
        // Dépôt Git
        REPO_URL   = 'https://github.com/JemAli211/mesut-ozil.git'
        BRANCH     = 'main'

        // Image Docker
        IMAGE_NAME = 'student-app'
        IMAGE_TAG  = '1.0'

        // Dossier Kubernetes
        K8S_DIR    = 'k8s'

        // 🔥 kubeconfig de ton utilisateur WSL
        KUBECONFIG = '/home/ali/.kube/config'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}",
                    credentialsId: 'github-token',
                    url: "${REPO_URL}"
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker build') {
            steps {
                sh '''
                    echo ">>> Build Docker image"
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .

                    echo ">>> Images Docker filtrées sur $IMAGE_NAME"
                    docker images | grep $IMAGE_NAME || true
                '''
            }
        }

        stage('Deployment Kubernetes') {
            steps {
                sh '''
                    echo ">>> Déploiement Kubernetes avec les manifests dans $K8S_DIR"

                    # MySQL
                    kubectl apply -f $K8S_DIR/mysql-secret.yaml --validate=false
                    kubectl apply -f $K8S_DIR/mysql-pv-pvc.yaml --validate=false
                    kubectl apply -f $K8S_DIR/mysql-deployment.yaml --validate=false
                    kubectl apply -f $K8S_DIR/mysql-service.yaml --validate=false

                    # Spring Boot
                    kubectl apply -f $K8S_DIR/spring-deployment.yaml --validate=false
                    kubectl apply -f $K8S_DIR/spring-service.yaml --validate=false

                    echo ">>> Etat des pods :"
                    kubectl get pods

                    echo ">>> Etat des services :"
                    kubectl get svc

                    echo ">>> Secrets :"
                    kubectl get secrets
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline exécuté avec succès 🎉'
        }
        failure {
            echo '❌ Erreur dans le pipeline – regarde les logs de chaque stage.'
        }
    }
}
