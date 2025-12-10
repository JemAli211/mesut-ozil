pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }

    environment {
        IMAGE_NAME = "alijemai/student-app"
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

        stage('Docker build') {
            steps {
                sh """
                    echo '>>> Build image Docker'
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker images | grep ${IMAGE_NAME} || true
                """
            }
        }

        // Si tu veux push sur DockerHub (facultatif, mais bien pour le workshop)
        /*
        stage('Docker push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKER_USER',
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }
        */

        stage('Deployment Kubernetes') {
            steps {
                sh """
                    echo '>>> kubectl apply sur les manifests'
                    kubectl apply -f ${K8S_DIR}/mysql-secret.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-pv-pvc.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-deployment.yaml
                    kubectl apply -f ${K8S_DIR}/mysql-service.yaml

                    kubectl apply -f ${K8S_DIR}/spring-deployment.yaml
                    kubectl apply -f ${K8S_DIR}/spring-service.yaml

                    echo '>>> Vérification'
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
