pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "nk779"
        DOCKER_IMAGE = "jenkins_devops_exams"// "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                    sh '''
                        echo "Let's run docker build"
                    '''
                }
            }
        }

        stage(' Docker run'){ // run container from our built image
            steps {
                script {
                    sh '''
                        echo "Let's run docker run"
                    '''
                }
            }
        }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                script {
                    sh '''
                        echo "Let's run test"
                    '''
                }
            }
        }

        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment{
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {
                script {
                    sh '''
                        #docker login -u $DOCKER_ID -p $DOCKER_PASS
                        #docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deployment in dev'){
            environment{
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                        echo "Let's deploy in dev"
                    '''
                }
            }
        }

        stage('Deployment on QA'){
            environment{
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                        echo "Let's deploy in qa"
                    '''
                }
            }
        }

        stage('Deployment on staging'){
            environment{
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
                script {
                    sh '''
                        echo "Let's deploy in staging"
                    '''
                }
            }
        }

        stage('Deployment on prod'){
            environment{
                KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins
            }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }

                script {
                    sh '''
                        echo "Let's deploy in prod"
                    '''
                }
            }
        }
    }

    post { // send email when the job has failed
        failure {
            echo "This will run if the job failed"
            mail to: "th_nguyen@gmx.de",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
