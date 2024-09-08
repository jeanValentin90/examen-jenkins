pipeline {
    environment { 
        DOCKER_ID = "jeanvalentin90" 
        DOCKER_IMAGE_CAST = "examen-jenkins_cast_service"
        DOCKER_IMAGE_MOVIE = "examen-jenkins_movie_service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any 
    stages {
        stage('Docker Build Cast Service'){ 
            steps {
                script {
                    sh '''
                    cd cast-service
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker Push Cast Service'){ 
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Docker Build Movie Service'){ 
            steps {
                script {
                    sh '''
                    cd movie-service
                    docker rm -f jenkins-movie
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG .
                    sleep 6
                    '''
                }
            }
        }
        stage('Docker Push Movie Service'){ 
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                script {
                    sh '''
                    docker login -u $DOCKER_ID -p $DOCKER_PASS
                    docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage('Deployment to ej-dev'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-dev.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade ej-microservices helm/ --values=values.yml -n ej-dev
                    '''
                }
            }
        }
        stage('Deployment to ej-staging'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-staging.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade ej-microservices helm/ --values=values.yml -n ej-staging
                    '''
                }
            }
        }
        stage('Deployment to ej-prod'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in ej-production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp helm/values-prod.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade ej-microservices helm/ --values=values.yml -n ej-prod
                    '''
                }
            }
        }
    }
    post {
        failure {
            echo "This will run if the job failed."
            mail to: "jean.valentin@outlook.com",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
