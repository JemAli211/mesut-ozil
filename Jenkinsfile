pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }

    environment {
        IMAGE_NAME = "alijemai/student-app"
        IMAGE_TAG  = "1.0"
        // adapte ce chemin si ton dossier k8s n'est pas ici
        K8S_DIR    = "student-management/k8s"
    }

      stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/JemAli211/mesut-ozil.git'
            }
        }


        stage('Build Maven') {
            steps {
                sh """
                    echo '>>> Build Maven dans student-management'
                    mvn -f student-management/pom.xml clean package -DskipTests
                """
            }
        }

        stage('Docker build') {
            steps {
                sh """
                    echo '>>> Build image Docker'
                    cd student-management
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    echo '>>> Images locales :'
                    docker images | grep ${IMAGE_NAME} || true
                """
            }
        }

        stage('Deployment Kubernetes') {
            steps {
                sh """
                    echo '>>> kubectl apply sur les manifests'

                    kubectl apply -f ${K8S_DIR}/mysql-secret.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-pv-pvc.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-deployment.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/mysql-service.yaml --validate=false

                    kubectl apply -f ${K8S_DIR}/spring-deployment.yaml --validate=false
                    kubectl apply -f ${K8S_DIR}/spring-service.yaml --validate=false

                    echo '>>> Vérification'
                    kubectl get pods
                    kubectl get svc
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline exécuté avec succès 🎉"
        }
        failure {
            echo "❌ Erreur dans le pipeline"
        }
    }
}
