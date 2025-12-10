pipeline {
    agent any
    tools {
        maven 'M2_HOME'
        jdk   'JAVA_HOME'
    }
    environment {
        IMAGE_NAME = "alijemai/student-app"
        IMAGE_TAG  = "1.0"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/JemAli211/mesut-ozil.git'
                
                script {
                    echo '>>> Vérification de la structure du projet'
                    sh 'pwd'
                    sh 'ls -la'
                    sh 'find . -name pom.xml'
                }
            }
        }
        stage('Build Maven') {
            steps {
                script {
                    echo '>>> Build Maven'
                    // Chercher le pom.xml et build depuis le bon répertoire
                    sh '''
                        if [ -f pom.xml ]; then
                            echo "pom.xml trouvé à la racine"
                            mvn clean package -DskipTests
                        elif [ -f student-management/pom.xml ]; then
                            echo "pom.xml trouvé dans student-management/"
                            cd student-management
                            mvn clean package -DskipTests
                        else
                            echo "ERREUR: pom.xml non trouvé!"
                            exit 1
                        fi
                    '''
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    echo '>>> Build image Docker'
                    sh '''
                        if [ -f Dockerfile ]; then
                            echo "Dockerfile trouvé à la racine"
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        elif [ -f student-management/Dockerfile ]; then
                            echo "Dockerfile trouvé dans student-management/"
                            cd student-management
                            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        else
                            echo "ERREUR: Dockerfile non trouvé!"
                            exit 1
                        fi
                        echo '>>> Images Docker locales :'
                        docker images | grep ${IMAGE_NAME} || true
                    '''
                }
            }
        }
        stage('Deployment Kubernetes') {
            steps {
                script {
                    echo '>>> Déploiement sur Kubernetes'
                    sh '''
                        # Trouver le répertoire k8s
                        if [ -d k8s ]; then
                            K8S_PATH="k8s"
                        elif [ -d student-management/k8s ]; then
                            K8S_PATH="student-management/k8s"
                        else
                            echo "ERREUR: Répertoire k8s non trouvé!"
                            exit 1
                        fi
                        
                        echo ">>> Application des manifests depuis $K8S_PATH"
                        kubectl apply -f $K8S_PATH/mysql-secret.yaml --validate=false
                        kubectl apply -f $K8S_PATH/mysql-pv-pvc.yaml --validate=false
                        kubectl apply -f $K8S_PATH/mysql-deployment.yaml --validate=false
                        kubectl apply -f $K8S_PATH/mysql-service.yaml --validate=false
                        kubectl apply -f $K8S_PATH/spring-deployment.yaml --validate=false
                        kubectl apply -f $K8S_PATH/spring-service.yaml --validate=false
                        
                        echo '>>> Vérification des ressources'
                        kubectl get pods
                        kubectl get svc
                    '''
                }
            }
        }
    }
    post {
        success {
            echo "✅ Pipeline exécuté avec succès 🎉"
        }
        failure {
            echo "❌ Erreur dans le pipeline"
        }
        always {
            echo ">>> Pipeline terminé"
        }
    }
}
