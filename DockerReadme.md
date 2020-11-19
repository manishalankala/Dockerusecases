
![image](https://user-images.githubusercontent.com/33985509/99667941-5e7d3d80-2a6d-11eb-95e5-de0aecb563a2.png)





curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"

sudo apt update

sudo apt install docker-ce

sudo systemctl is-active docker

sudo systemctl is-enabled docker

sudo systemctl status docker


sudo systemctl stop docker			#stop the docker service

sudo systemctl start docker			#start the docker service

sudo systemctl  restart docker		#restart the docker service

docker version

sudo usermod -aG docker username




## WORKDIR :

It is used for switch to that directory for copy or add files or run some command in that directory in container. It also helps to create the new directory, which are used for building the image, copy the code and settings to it.


## ENV 

It's use for environment variable of docker images. Docker image contains the lightweight operating system. we can push and override the environment variable from  calling it
COPY 

Helps to copy the file from host machine to docker container.


## ADD 

Helps to copy the file from host machine to docker image or container. It as like as copy instractions, it has extra functionality, it copy the file from remote also



## RUN 
 
 Run the command inside the docker image at image building time. It support valid operating system command.   executes command(s) in a new layer and creates a new image

## CMD 

sets default command and/or parameters, which can be overwritten from command line when docker container runs

## ENTRYPOINT 

used for configure the container that will run as an executable. We can add the multiple command shell file .sh file in entrypoint instructions. Also add the multiple command in this instruction

## EXPOSE 

It will help to expose the port for to in docker to access by other container or images. we also mapping with host computer port during run the docker container.

## version: 

contains docker compose file format version. There is several file format available form 1.x to 3.x.

## services: 

services blocks contains all the applications or services definition which is needed for run the applications successfully. for example if we want to run the web apps, we need a database and other services. so we can add multiple service or app definitions in services blocks.

## build: 

there is a lot of option for build instruction. we can build image from another image. we can provide the target folder to build in `build` block. build has some sub blocks, those are
     
## context:  

which will takes to a directory, which contains a Dockerfile, or a url to a git repository. where the value supplied with relative path, it will send to docker deamon to building process.


## dockerfile

if we want to specify or use different docker file to build docker image by docker compose file. we need to use it.


## command: 

Run the command in docker container from the docker-compose.yml file. it can run any valid os and os application command inside the container.

## volumes: 

we can mount the specific host path or named volume to docker contains which contains the media files and other files needed for run the application. for my apps i have to mount the /app folder to container.

## expose: 

provide port access to docker system so other container can access the service by this port. expose only publish the port within docker system, so host computer service or app can not get this port to access service.


## ports: 

it use for mapping the ports between HOST:DOCKERCONTAINER. other docker container service and host computer service can get access to that port, to get service information.


## environment: 

it will add and override the environment variable to docker container or image. which is then you can use it as environment variable to the specified docker container image.

## env_file: 

we can add environment variable file there. we can use it also define the or override the environment variable of the docker container or image. we can add the multiple env file in env_file block.


## depends_on: 

developer can add the dependency service, which is needed for run the or build image in this section. it will ensure all the dependency service are available before start build the docker images or service.

## networks: 

it use in two context. one is in the service definition. other is used for create the network in docker environment. network in side the service definition will assing the network to the service.
