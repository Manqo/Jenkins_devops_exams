pipeline {
    agent any

    environment {
        DOCKER_USER = 'manqo' // Hier deinen Namen eintragen!
        IMAGE_NAME = "jenkins-exam-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Baut das Image
                    sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    
                    // Loggt sich bei DockerHub ein (Nutzt die ID 'dockerhub-creds' aus Jenkins)
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh "kubectl apply -f k8s-deploy.yaml -n dev"
                // Optional: Falls du Helm nutzt, wäre der Befehl:
                // sh "helm upgrade --install my-app ./charts --namespace dev"
            }
        }

        stage('Deploy to QA & Staging') {
            steps {
                sh "kubectl apply -f k8s-deploy.yaml -n qa"
                sh "kubectl apply -f k8s-deploy.yaml -n staging"
            }
        }

        stage('Promote to Prod (Manual)') {
            when {
                branch 'master' // Nur wenn du im Master-Branch arbeitest
            }
            steps {
                // Das ist der manuelle Stop für die Prüfung!
                input message: "Soll die Anwendung nach PRODUKTION ausgerollt werden?"
                
                sh "kubectl apply -f k8s-deploy.yaml -n prod"
            }
        }
    }
}
