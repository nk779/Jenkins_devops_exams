pipeline {
    environment { // Declaration of environment variables
        DOCKER_ID = "nk779"
        DOCKER_MOVIE_IMAGE = "movie-service"
        DOCKER_CAST_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    
    agent any // Jenkins will be able to select all available agents
    stages {
        stage('Docker Build'){ // docker build image stage
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG ./movie-service
                        sleep 6
                    '''
                }
                script {
                    sh '''
                        docker build -t $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG ./cast-service
                        sleep 6
                    '''
                }
            }
        }

        stage(' Docker run'){ // run 'docker network create -d bridge jk_exam'
            steps {
                script {
                    sh '''
                        echo "Run cast application: cast-db & cast-service"
                        docker run --rm -d --name cast-db --network jk_exam --env POSTGRES_USER=cast_db_username --env POSTGRES_PASSWORD=cast_db_password --env POSTGRES_DB=cast_db_dev postgres:12.1-alpine
                        docker run --rm -d -p 8002:8000 --name cast_service --network jk_exam --env DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast-db/cast_db_dev $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG  uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                        
                        echo "Run movie application: movie-db & movie-service with injection cast-service"
                        docker run --rm -d --name movie-db --network jk_exam --env POSTGRES_USER=movie_db_username --env POSTGRES_PASSWORD=movie_db_password --env POSTGRES_DB=movie_db_dev postgres:12.1-alpine
                        docker run --rm -d -p 8001:8000 --name movie_service --network jk_exam --env DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie-db/movie_db_dev --env CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                    
                        echo "Start nginx reverse proxy"
                        docker run --rm -d -p 8088:8000 --name nginx-proxy --network jk_exam -v ./nginx_config.conf:/etc/nginx/conf.d/default.conf nginx
                    '''
                }
            }
        }

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                script {
                    sh '''
                        echo "Let's run test"
                        nginx_ip=$(docker network inspect -f '{{json .Containers}}' jk_exam | jq '..|if type == "object" and has("Name") then select(.Name=="nginx-proxy") | .IPv4Address else empty end' -r | awk -F / '{print $1}')
                        movie_rq_status_code=$(curl --write-out %{http_code} --silent --output /dev/null ${nginx_ip}:8088/api/v1/movie/docs)
                        cast_rq_status_code=$(curl --write-out %{http_code} --silent --output /dev/null ${nginx_ip}:8088/api/v1/casts/docs)
 
                        if [[ "$movie_rq_status_code" -ne 200 ]] || [ "$statcast_rq_status_codeus_code_2" -ne 200 ]]; then
                            echo "curl ${nginx_ip}:8088/api/v1/movie/docs: $movie_rq_status_code"
                            echo "curl ${nginx_ip}:8088/api/v1/casts/docs: $cast_rq_status_code" 
                            error "Test failed!"
                        else
                            "Test passed!"
                        fi
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
                        docker login -u $DOCKER_ID -p $DOCKER_PASS
                        docker push $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG
                        docker push $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG
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
        success {
            sh '''
                docker stop nginx-proxy movie_service movie-db cast_service cast-db
                docker rmi $DOCKER_ID/$DOCKER_MOVIE_IMAGE:$DOCKER_TAG $DOCKER_ID/$DOCKER_CAST_IMAGE:$DOCKER_TAG 
            '''
        }

        failure {
            echo "This will run if the job failed"
            mail to: "th_nguyen@gmx.de",
                subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
    }
}
