


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
