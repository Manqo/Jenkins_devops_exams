pipeline {
    agent any

    environment {
        DOCKER_USER = 'manqo' 
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
                    sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${IMAGE_NAME}:latest"
                    
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
                sh "kubectl apply -f k8s-deploy.yaml -n dev"
            }
        }

        stage('Deploy to QA & Staging') {
            steps {
                sh "kubectl apply -f k8s-deploy.yaml -n qa"
                sh "kubectl apply -f k8s-deploy.yaml -n staging"
            }
        }

        stage('Promote to Prod (Manual)') {
            steps {
                input message: "Soll die Anwendung nach PRODUKTION ausgerollt werden?"
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
