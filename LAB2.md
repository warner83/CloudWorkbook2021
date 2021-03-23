# Install Docker
  sudo apt install apt-transport-https ca-certificates curl software-properties-common
  
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
  
  sudo apt install docker-ce
  
# Post Installation
  sudo systemctl status docker
  
# Run a container
  docker run hello-world
  
# Repository
  docker run hello-world
  
# Download and run an image
  docker pull ubuntu
  docker run -it ubuntu
  
# List containers
  docker ps –a
  docker rm ID
  
# List images
  docker images
  docker rmi ID
  
# Define a custom image

  # Dockerfile:
  
  FROM ubuntu
  RUN apt-get update && apt-get install -y python3
  ADD ./my_program.py /root/
  CMD ["/usr/bin/python3", "/root/my_program.py"]
  
  docker build -t custom-ubuntu .
  docker run custom-ubuntu
  
# Expose a service
  # Dockerfile:
  FROM ubuntu
  RUN apt-get update && apt-get install -y nginx
  EXPOSE 80
  CMD ["nginx", "-g", "daemon off;"]
  
  docker run -p 80:80 nginx-ubuntu
  curl http://127.0.0.1/
  
# Run in background
  docker run -p 80:80 -d nginx-Ubuntu
  docker stop ID
  
# Communication among containers
  docker network inspect bridge
