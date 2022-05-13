# docker-compose-in-jenkins
Configure docker image to run as Jenkins server and to run docker and docker-compose commands inside the container.  

Steps: 

Open up a terminal window. 
Create a bridge network in Docker using the following docker network create command: 
  [root] docker network create jenkins 

In order to execute Docker commands inside Jenkins nodes, download and run the docker:dind Docker image using the following docker run command: 
  [root] docker run --name jenkins-docker --rm --detach --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home --publish 2376:2376 docker:dind --storage-driver overlay2 
  
Customize the official Jenkins Docker image by executing the two steps: 
  Create Dockerfile with the following content: 

    FROM jenkins/jenkins:2.332.3-jdk11 
    USER root 
    RUN apt-get update && apt-get install -y lsb-release 
    RUN apt-get update -qq && apt-get install -qqy \ 
        apt-transport-https \ 
        ca-certificates \ 
        curl \ 
        lxc \ 
        iptables \ 
        vim \ 
        sudo \ 
        apt-utils  
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \ 
      https://download.docker.com/linux/debian/gpg 
    RUN echo "deb [arch=$(dpkg --print-architecture) \ 
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \ 
      https://download.docker.com/linux/debian \ 
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list                     
    RUN apt-get update && apt-get install -y docker-ce-cli 
    RUN curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-    `uname -s`-`uname -m` > ~/docker-compose 
    RUN chmod +x ~/docker-compose 
    RUN sudo mv ~/docker-compose /usr/local/bin/docker-compose 
    RUN docker-compose --version 
    USER jenkins 

Build a new docker image from this Dockerfile and assign the image a meaningful name, e.g. "myjenkins-blueocean:2.332.3-1":  
  [root] docker build -t myjenkins-blueocean:2.332.3-1 . 

Run your own myjenkins-blueocean:2.332.3-1 image as a container in Docker using the following docker run command: 
  [root] docker run --name jenkins-blueocean --restart=on-failure --detach --network jenkins --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --publish 49000:8080 --publish 50000:50000 --volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro myjenkins:2.332.3-1 

You can access this container using the command below: 
  [root] docker exec -it jenkins-blueocean bash 
  [jenkins@61f1f7705996] docker --version 
  o/p=>Docker version 20.10.16, build aa7e414 
  [jenkins@61f1f7705996] docker-compose --version 
  o/p=>docker-compose version 1.29.2, build 5becea4c 

Unlock Jenkins 
  Browse to http://localhost:49000 

Reference: 
Docker (jenkins.io) 

 
