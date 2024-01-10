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
  
 




 