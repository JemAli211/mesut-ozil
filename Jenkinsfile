pipeline {
    agent any

    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/JemAli211/mesut-ozil.git'
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }
stage('Build Docker Image') {
    steps {
        script {
            sh 'docker build -t alijemai/connect-sphere:latest .'
        }
    }
}

stage('Push Docker Image') {
    steps {
        script {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                             usernameVariable: 'DOCKER_USER',
                                             passwordVariable: 'DOCKER_PASS')]) {

                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                sh "docker push alijemai/connect-sphere:latest"
            }
        }
    }
}

    }

    post {
        success {
            echo "Build réussi 🎉"
        }
        failure {
            echo "Build échoué ❌"
        }
    }
}
