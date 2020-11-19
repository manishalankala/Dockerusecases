


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

--> It will help to expose the port for to in docker to access by other container or images. we also mapping with host computer port during run the docker container.
