pipeline {
    agent any
    environment {
        DOCKER_COMPOSE_FILE = "${WORKSPACE}/docker-compose.yml"
        DOCKER_ID = "elie150100"
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
                        docker push ${MOVIE_SERVICE_IMAGE}:v.7.0
                        docker push ${CAST_SERVICE_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} up -d"
            }
        }
        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                        s helm upgrade --install movie-service ./movie-service \
                            -f values.yaml \
                            -f movie-service/values.yaml \
                            --namespace production
                            
                            helm upgrade --install cast-service ./cast-service \
                            -f values.yaml \
                            -f cast-service/values.yaml \
                            --namespace production
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
