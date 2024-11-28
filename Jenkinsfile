pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = "master/docker-compose.yml"
        DOCKER_ID = "eliedatasctst"
        MOVIE_SERVICE_IMAGE = "${DOCKER_ID}/movie-service"
        CAST_SERVICE_IMAGE = "${DOCKER_ID}/cast-service"
        DOCKER_TAG = "v.${BUILD_NUMBER}.0"
        DOCKER_CREDENTIALS = credentials('DOCKER_HUB_PASS')
    }
    
    stages {
        stage('Build and Push Images') {
            steps {
                sh "docker-compose -f ${DOCKER_COMPOSE_FILE} build"
                
                script {
                    docker.withRegistry('', DOCKER_CREDENTIALS) {
                        sh """
                            docker tag movie_service ${MOVIE_SERVICE_IMAGE}:${DOCKER_TAG}
                            docker tag cast_service ${CAST_SERVICE_IMAGE}:${DOCKER_TAG}
                            docker push ${MOVIE_SERVICE_IMAGE}:${DOCKER_TAG}
                            docker push ${CAST_SERVICE_IMAGE}:${DOCKER_TAG}
                        """
                    }
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
                        sleep 30
                        curl localhost:8080/api/v1/movies/docs 
                        curl localhost:8080/api/v1/casts/docs 
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
        helm upgrade --install app fastapi --values=values.yml --namespace ${namespace}
    """
}
