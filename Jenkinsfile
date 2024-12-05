pipeline {
    agent any
    environment {
        DOCKER_COMPOSE_FILE = "${WORKSPACE}/docker-compose.yml"
        DOCKER_ID = "elie150100"
        MOVIE_SERVICE_IMAGE = "${DOCKER_ID}/movie-service"
        CAST_SERVICE_IMAGE = "${DOCKER_ID}/cast-service"
        DOCKER_TAG = "v.${BUILD_NUMBER}.0"
        DOCKER_CREDENTIALS = credentials('DOCKER_HUB_PASS')
        KUBECONFIG = credentials("config") // Secret file kubeconfig
    }
    stages {
        stage('Deploy to Kubernetes') {
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
        # Préparer la configuration Kubernetes
        rm -Rf .kube
        mkdir .kube
        cat \$KUBECONFIG > .kube/config
        
        # Déployer le service Movie
        cp movie-service/values.yaml values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        kubectl get namespace ${namespace} || kubectl create namespace ${namespace}
        helm upgrade --install movie-service ./movie-service --values=values.yml --namespace ${namespace}

        # Déployer le service Cast
        cp cast-service/values.yaml values.yml
        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        helm upgrade --install cast-service ./cast-service --values=values.yml --namespace ${namespace}
    """
}
