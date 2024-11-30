pipeline {
    agent any
    environment {
        DOCKER_COMPOSE_FILE = "${WORKSPACE}/docker-compose.yml"
        DOCKER_ID = "eliedatasctst"
        MOVIE_SERVICE_IMAGE = "${DOCKER_ID}/movie-service"
        CAST_SERVICE_IMAGE = "${DOCKER_ID}/cast-service"
        DOCKER_TAG = "v.${BUILD_NUMBER}.0"
        DOCKER_CREDENTIALS = credentials('DOCKER_HUB_PASS')
    }
    stages {
        stage('Check Workspace') {
            steps {
                sh "pwd"
                sh "ls -al"
            }
        }
        stage('Build and Push Images') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} build"
                script {
                    sh """
                        docker login -u ${DOCKER_ID} -p ${DOCKER_CREDENTIALS}
                        docker tag movie_service ${MOVIE_SERVICE_IMAGE}:${DOCKER_TAG}
                        docker tag cast_service ${CAST_SERVICE_IMAGE}:${DOCKER_TAG}
                        docker push ${MOVIE_SERVICE_IMAGE}:${DOCKER_TAG}
                        docker push ${CAST_SERVICE_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
<<<<<<< HEAD
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} up -d"
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        sleep 30
                        curl -s localhost:8080/api/v1/movies/docs || echo "Movies service unavailable"
                        curl -s localhost:8080/api/v1/casts/docs || echo "Casts service unavailable"
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = credentials("config")
            }
            stages {
                stage('Deploy to Dev') {
                    steps {
                        script {
                            deployToK8s('dev')
                        }
                    }
                }
                stage('Deploy to QA') {
                    steps {
                        script {
                            deployToK8s('qa')
                        }
                    }
                }
                stage('Deploy to Staging') {
                    steps {
                        script {
                            deployToK8s('staging')
                        }
                    }
                }
                stage('Deploy to Prod') {
                    steps {
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to deploy in production?', ok: 'Yes'
                        }
                        script {
                            deployToK8s('prod')
                        }
                    }
                }
            }
        }
    }
}

def deployToK8s(String namespace) {
    sh """
        rm -Rf .kube
        mkdir .kube
        cat \$KUBECONFIG > .kube/config
        cp fastapi/values.yaml values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        kubectl get namespace ${namespace} || kubectl create namespace ${namespace}
        helm upgrade --install app fastapi --values=values.yml --namespace ${namespace}
    """
}
=======
                // Déployer les services en utilisant le fichier docker-compose.yml
                sh "docker-compose -f $DOCKER_COMPOSE_FILE up -d"
            }
        }
        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
                    steps {
                            script {
                            sh '''
                            curl localhost:8081/api/v1/movies/docs 
                            curl localhost:8081/api/v1/casts/docs 
                            '''
                            }
                    }
                }
 stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {
                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                '''
                }
            }

        }
 
 
 
        stage('dev'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp exemple/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app fastapi --values=values.yml --namespace dev
                '''
                }
            }

        }
        stage('Deploiement en QA') {
    environment {
        KUBECONFIG = credentials("config") // on récupère le kubeconfig depuis le fichier secret "config" enregistré sur Jenkins
    }
    steps {
        script {
            sh '''
            rm -Rf .kube
            mkdir .kube
            ls
            cat $KUBECONFIG > .kube/config
            cp fastapi/values.yaml values.yml
            cat values.yml
            sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
            helm upgrade --install app fastapi --values=values.yml --namespace qa
            '''
        }
    }
}
}
}
>>>>>>> Ajout de nouveaux fichiers
