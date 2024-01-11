# DATASCIENTEST JENKINS EXAM
# python-microservice-fastapi
Learn to build your own microservice using Python and FastAPI

## How to run??
 - Make sure you have installed `docker` and `docker-compose`
 - Run `docker-compose up -d`
 - Head over to http://localhost:8080/api/v1/movies/docs for movie service docs 
   and http://localhost:8080/api/v1/casts/docs for cast service docs

## Dev architecture
 - Run application with `docker-compose up` starts 5 containers:
    nginx(reverse-proxy to redirect to backend application)
    cast_service container
    postgres-db for cast_service
    movie_service container
    postgres-db for movie_service

 ## K8s architecture
 Principle transform the given docker-compose into K8s resource objects:

 K8s movie-service application: private, only available in Cluster intern
 - docker-compose.services.movieService => k8s.deployment with command, args and binding database by using env variables => on file: /kubernetes/jenkins-devops-exams/templates/deployments.yaml
 - docker-compose.services.movie_db => k8s.statefulset + k8s.persitentVolumeClaim => on file: /kubernetes/jenkins-devops-exams/templates/statefulset-svc.yaml
 - define a k8s.service object using ClusterIP => on file: /kubernetes/jenkins-devops-exams/templates/services.yaml

 K8s cast-service application: private, only available in Cluster intern
 - docker-compose.services.castService => k8s.deployment with command, args and binding database by using env variables => on file: /kubernetes/jenkins-devops-exams/templates/deployments.yaml
 - docker-compose.services.cast_db => k8s.statefulset + k8s.persitentVolumeClaim => on file: /kubernetes/jenkins-devops-exams/templates/statefulset-svc.yaml
 - define a k8s.service object using ClusterIP => on file: /kubernetes/jenkins-devops-exams/templates/services.yaml

 K8s nginx: public as a reverse proxy listening on port 8080 and loading request to the correct service 
 - docker-compose.services.nginx => k8s.deployment using ConfigMap to define nginx_config.config => on file: /kubernetes/jenkins-devops-exams/templates/deployments.yaml & ..configmap.yaml
 - define a k8s.service object using NodePort => on file: /kubernetes/jenkins-devops-exams/templates/services.yaml

 ## Helm install manually
 - cd kubernetes
 - helm lint ./jenkins-devops-exams/ # validate k8s resource object codes
 - helm upgrade -i jenkins-devops-exams-chart ./jenkins-devops-exams --values=./jenkins-devops-exams/values.yaml # the -i flag install the release if it doesn't exit

 ## Build Jenkinsfile: run locally all cmd of scripts to verify cmd
 - docker build -t nk779/movie-service:v1.0.0 ./movie-service
 - docker build -t nk779/cast-service:v1.0.0 ./cast-service

 - docker network create -d bridge jk_exam

 - docker run --rm -d --name cast-db --network jk_exam --env POSTGRES_USER=cast_db_username --env POSTGRES_PASSWORD=cast_db_password --env POSTGRES_DB=cast_db_dev postgres:12.1-alpine
 - docker run --rm -d --name movie-db --network jk_exam --env POSTGRES_USER=movie_db_username --env POSTGRES_PASSWORD=movie_db_password --env POSTGRES_DB=movie_db_dev postgres:12.1-alpine

 - docker run --rm -d -p 8002:8000 --name cast_service --network jk_exam --env DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast-db/cast_db_dev nk779/cast-service:v1.0.0  uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

 - docker run --rm -d -p 8001:8000 --name movie_service --network jk_exam --env DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie-db/movie_db_dev --env CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ nk779/movie-service:v1.0.0 uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

 // change your own absolut path to the file nginx_config.conf
 - docker run --rm -d -p 8080:8000 --name nginx-proxy --network jk_exam -v ~/DevOpsBootcamp/Projects/Jenkins_devops_exams/nginx_config.conf:/etc/nginx/conf.d/default.conf nginx

 
  
 




 