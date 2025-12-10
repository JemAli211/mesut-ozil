pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }

    environment {
        // Ton dépôt Git
        REPO_URL   = 'https://github.com/JemAli211/mesut-ozil.git'
        BRANCH     = 'main'

        // Nom + tag de l'image Docker locale (dans Minikube)
        IMAGE_NAME = 'student-app'
        IMAGE_TAG  = '1.0'

        // Très important : dossier k8s à la racine du repo
        K8S_DIR    = 'k8s'
    }

    stages {

        // 1) Récupération du code
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}",
                    credentialsId: 'github-token',
                    url: "${REPO_URL}"
            }
        }

        // 2) Build Maven (pom.xml à la racine du repo)
        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 3) Build de l'image Docker dans le moteur Docker de Minikube
        stage('Docker build (Minikube)') {
            steps {
                sh """
                    echo '>>> Activation du moteur Docker de Minikube'
                    eval \$(minikube docker-env)

                    echo '>>> Build de l\\'image Docker'
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    echo '>>> Images Docker filtrées sur ${IMAGE_NAME}'
                    docker images | grep ${IMAGE_NAME} || true
                """
            }
        }

        // 4) Déploiement sur Kubernetes avec les manifests dans k8s/
        stage('Deployment Kubernetes') {
            steps {
                sh """
                    echo '>>> kubectl apply sur les manifests'

                    # MySQL
                    kubectl apply -f ${K8S_DIR}/mysql-secret.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-pv-pvc.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-deployment.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-service.yaml --validate=false

                    # Spring Boot
                    kubectl apply -f ${K8S_DIR}/spring-deployment.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/spring-service.yaml --validate=false

                    echo '>>> Etat des ressources :'
                    kubectl get pods
                    kubectl get svc
                    kubectl get secrets
                """
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
