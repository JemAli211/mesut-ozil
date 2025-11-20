pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk11'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    credentialsId: 'github-token',
                    url: 'https://github.com/JemAli211/mesut-ozil.git'
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
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
