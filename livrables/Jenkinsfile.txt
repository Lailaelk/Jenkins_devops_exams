pipeline {

    agent any

    environment {

        DOCKER_USER = "lolaelk"

        MOVIE_IMAGE = "lolaelk/movie-service"
        CAST_IMAGE  = "lolaelk/cast-service"

        TAG = "${BUILD_NUMBER}"
    }


    stages {


        stage('Checkout Code') {

            steps {

                git branch: 'master',
                    url: 'https://github.com/Lailaelk/Jenkins_devops_exams.git'

            }
        }



        stage('Build Docker Images') {

            steps {

                sh '''

                echo "Building Movie Service image..."

                docker build \
                -t ${MOVIE_IMAGE}:${TAG} \
                ./movie-service



                echo "Building Cast Service image..."

                docker build \
                -t ${CAST_IMAGE}:${TAG} \
                ./cast-service

                '''
            }
        }




        stage('Login DockerHub') {

            steps {

                withCredentials([

                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )

                ]) {


                    sh '''

                    echo $DOCKER_PASSWORD | docker login \
                    -u $DOCKER_USERNAME \
                    --password-stdin

                    '''

                }
            }
        }





        stage('Push Docker Images') {

            steps {

                sh '''

                docker push ${MOVIE_IMAGE}:${TAG}

                docker push ${CAST_IMAGE}:${TAG}

                '''

            }
        }





        stage('Create Kubernetes Namespaces') {

            steps {

                sh '''

                kubectl create namespace dev || true

                kubectl create namespace qa || true

                kubectl create namespace staging || true

                kubectl create namespace prod || true

                '''

            }
        }






        stage('Deploy DEV') {

            steps {

                sh '''

                helm upgrade --install app-dev ./charts \
                -n dev \
                -f charts/values-dev.yaml \
                --set movie.image.tag=${TAG} \
                --set cast.image.tag=${TAG}

                '''

            }
        }





        stage('Deploy QA') {

            steps {

                sh '''

                helm upgrade --install app-qa ./charts \
                -n qa \
                -f charts/values-qa.yaml \
                --set movie.image.tag=${TAG} \
                --set cast.image.tag=${TAG}

                '''

            }
        }






        stage('Deploy STAGING') {

            steps {

                sh '''

                helm upgrade --install app-staging ./charts \
                -n staging \
                -f charts/values-staging.yaml \
                --set movie.image.tag=${TAG} \
                --set cast.image.tag=${TAG}

                '''

            }
        }





        stage('Approval for Production') {

            steps {

                input message: 'Deploy to Production?',
                      ok: 'Deploy'

            }

        }






        stage('Deploy PROD') {

            steps {

                sh '''

                helm upgrade --install app-prod ./charts \
                -n prod \
                -f charts/values-prod.yaml \
                --set movie.image.tag=${TAG} \
                --set cast.image.tag=${TAG}

                '''

            }

        }


    }





    post {


        success {

            echo '✅ Pipeline completed successfully'

        }


        failure {

            echo '❌ Pipeline failed - check logs'

        }

    }

}
