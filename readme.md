
## YouTube Link（refer:https://github.com/devopsjourney1/jenkins-101）
For the full 1 hour course watch out youtube:
https://www.youtube.com/watch?v=6YZvp2GwT0A

# Installation
## Build the Jenkins BlueOcean Docker Image by https://www.jenkins.io/doc/book/installing/docker/#setup-wizard
1. Open up a terminal window.
2. Create a bridge network in Docker using the following docker network create command:
```
docker network create jenkins

```
## In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image
```
docker run \
  --name jenkins-docker \
  --rm \
  --detach \
  --privileged \
  --network jenkins \
  --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind \
  --storage-driver overlay2
```
## Create the network 'jenkins'
```
docker network create jenkins
```
### 3. Customize the official Jenkins Docker image, by executing the following two steps:
### a. Create a Dockerfile with the following 
```
FROM jenkins/jenkins:2.414.1-jdk17
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
```
### b. Build a new docker image from this Dockerfile, and assign the image a meaningful name, such as "myjenkins-blueocean:2.414.1-1":
```
docker build -t myjenkins-blueocean:2.414.1-1 .
```
### Run your own myjenkins-blueocean:2.414.1-1 image as a container in Docker using the following docker run command:
```
docker run \
  --name jenkins-blueocean \
  --restart=on-failure \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 \
  --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.414.1-1

```
## 3. check logs to get password for unlock Jenkins(with your container id)
### This may also be found at: /var/jenkins_home/secrets/initialAdminPassword:
```
$ docker logs <your jenkins-blueocean id>
```

## Connect to the Jenkins
```
https://localhost:8080/

```
