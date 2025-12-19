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

        // Dossier des manifests Kubernetes (dossier k8s à la racine du repo)
        K8S_DIR    = 'k8s'

        // 🔥 kubeconfig à utiliser pour kubectl
        // (on a vérifié : /home/ali/.kube/config existe bien)
        KUBECONFIG = '/home/ali/.kube/config'
    }

    stages {

        // 1) Récupération du code source
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

      stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
            sh '''
              mvn -B -DskipTests sonar:sonar \
                -Dsonar.projectKey=student-management \
                -Dsonar.projectName=student-management
            '''
        }
    }
}


        // 3) Build de l'image Docker avec le Dockerfile à la racine
        stage('Docker build') {
            steps {
                script {
                    echo ">>> Build de l'image Docker"
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    
                    echo ">>> Images Docker filtrées sur ${IMAGE_NAME}"
                    sh "docker images | grep ${IMAGE_NAME} || true"
                }
            }
        }

        // 4) Déploiement sur Kubernetes avec kubectl
        stage('Deployment Kubernetes') {
            steps {
                script {
                    echo ">>> Dépploiement Kubernetes avec les manifests dans ${K8S_DIR}"
                    sh 'echo "KUBECONFIG utilisé : $KUBECONFIG"'

                    // MySQL
                    sh "kubectl apply -f ${K8S_DIR}/mysql-secret.yaml --validate=false"
                    sh "kubectl apply -f ${K8S_DIR}/mysql-pv-pvc.yaml --validate=false"
                    sh "kubectl apply -f ${K8S_DIR}/mysql-deployment.yaml --validate=false"
                    sh "kubectl apply -f ${K8S_DIR}/mysql-service.yaml --validate=false"

                    // Spring Boot
                    sh "kubectl apply -f ${K8S_DIR}/spring-deployment.yaml --validate=false"
                    sh "kubectl apply -f ${K8S_DIR}/spring-service.yaml --validate=false"

                    echo ">>> Etat des pods :"
                    sh "kubectl get pods"

                    echo ">>> Etat des services :"
                    sh 'kubectl get svc'

                    echo ">>> Secrets :"
                    sh "kubectl get secrets"
                }
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
