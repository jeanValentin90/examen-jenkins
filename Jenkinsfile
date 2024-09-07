pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "jeanvalentin90" // replace this with your docker-id
DOCKER_IMAGE_CAST = "examen-jenkins_cast_service"
DOCKER_IMAGE_MOVIE = "examen-jenkins_movie_service"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {
        stage(' Docker Build Cast Service'){ // docker build image stage
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
        stage('Docker Push  Cast Service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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

stages {
        stage(' Docker Build movie Service'){ // docker build image stage
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
        stage('Docker Push  Cast Service'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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

stage('Deploiement en ej-dev'){
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
                cp helm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ej-microservices --values=values.yml --namespace ej-dev
                '''
                }
            }
        }
stage('Deploiement en ej-staging'){
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
                cp helm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ej-microservices --values=values.yml --namespace ej-staging
                '''
                }
            }
        }
  stage('Deploiement en ej-prod'){
        environment
        {
        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
        }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on ej-production environment
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in ej-production ?', ok: 'Yes'
                    }

                script {
                sh '''
                rm -Rf .kube
                mkdir .kube
                ls
                cat $KUBECONFIG > .kube/config
                cp helm/values.yaml values.yml
                cat values.yml
                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                helm upgrade --install app ej-microservices --values=values.yml --namespace ej-prod
                '''
                }
            }
        }

}
post { // send email when the job has failed
    // ..
    failure {
        echo "This will run if the job failed"
        mail to: "jean.valentin@outlook.com",
             subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
             body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
    }
    // ..
}
}
