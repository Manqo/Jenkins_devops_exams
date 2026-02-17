pipeline {
    agent any

    environment {
        // Ersetze 'manqo' durch deinen tatsächlichen DockerHub-Namen, falls er anders ist
        DOCKER_USER = 'manqo' 
        IMAGE_NAME = "jenkins-exam-app"
        // Nutzt die Jenkins Build-Nummer als Tag für DockerHub
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Holt den Code von deinem GitHub Repository
                checkout scm
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // 1. Build: Erstellt das Image aus dem Dockerfile im aktuellen Verzeichnis
                    sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    
                    // 2. Push: Loggt sich mit den in Jenkins gespeicherten Credentials ein und schiebt das Image hoch
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                // Rollout in den Namespace 'dev'
                sh "kubectl apply -f k8s-deploy.yaml -n dev"
            }
        }

        stage('Deploy to QA & Staging') {
            steps {
                // Rollout in die Test-Umgebungen
                sh "kubectl apply -f k8s-deploy.yaml -n qa"
                sh "kubectl apply -f k8s-deploy.yaml -n staging"
            }
        }

        stage('Promote to Prod (Manual)') {
            // Hinweis: 'when' wurde entfernt, damit der Button bei JEDEM Build auf 'master' erscheint
            steps {
                // Dieser Befehl pausiert die Pipeline und zeigt den Button in Jenkins an
                input message: "Soll die Anwendung nach PRODUKTION ausgerollt werden?"
                
                // Rollout in den finalen Namespace
                sh "kubectl apply -f k8s-deploy.yaml -n prod"
            }
        }
    }

    post {
        success {
            echo 'Pipeline erfolgreich abgeschlossen!'
        }
        failure {
            echo 'Pipeline ist fehlgeschlagen. Bitte Console Output prüfen.'
        }
    }
}
