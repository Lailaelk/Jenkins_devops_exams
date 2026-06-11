pipeline {
    agent any

    environment {
        IMAGE_NAME = "lolaelk/fastapi-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Lailaelk/Jenkins_devops_exams.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $IMAGE_NAME:$TAG ."
            }
        }

        stage('Login DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push $IMAGE_NAME:$TAG"
            }
        }

        stage('Create Namespaces') {
            steps {
                sh """
                kubectl create namespace dev || true
                kubectl create namespace qa || true
                kubectl create namespace staging || true
                kubectl create namespace prod || true
                """
            }
        }

        stage('Deploy DEV') {
            when { branch "main" }
            steps {
                sh """
                helm upgrade --install app-dev ./charts \
                -n dev \
                -f charts/values-dev.yaml \
                --atomic \
                --set image.tag=$TAG
                """
            }
        }

        stage('Deploy QA') {
            when { branch "main" }
            steps {
                sh """
                helm upgrade --install app-qa ./charts \
                -n qa \
                -f charts/values-qa.yaml \
                --atomic \
                --set image.tag=$TAG
                """
            }
        }

        stage('Deploy STAGING') {
            when { branch "main" }
            steps {
                sh """
                helm upgrade --install app-staging ./charts \
                -n staging \
                -f charts/values-staging.yaml \
                --atomic \
                --set image.tag=$TAG
                """
            }
        }

        stage('Approval for Production') {
            when { branch "main" }
            steps {
                input message: "⚠️ Deploy to PRODUCTION? Manual approval required."
            }
        }

        stage('Deploy PROD') {
            when { branch "main" }
            steps {
                sh """
                helm upgrade --install app-prod ./charts \
                -n prod \
                -f charts/values-prod.yaml \
                --atomic \
                --set image.tag=$TAG
                """
            }
        }
    }

    post {
        success {
            echo "✅ SUCCESS - Pipeline completed successfully"
        }
        failure {
            echo "❌ FAILED - Check logs for errors"
        }
    }
}
