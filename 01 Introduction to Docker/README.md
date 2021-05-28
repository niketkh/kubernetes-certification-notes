Introduction to Docker
----------------------

What is Docker?
What are the challenges it solves?



Commands
--------

Running within CentOS VM

-> sudo yum remove docker docker-common docker-selinux docker-engine
-> sudo yum install -y yum-utils device-mapper-persistent-data lvm2
-> sudo yun install docker-ce
-> sudo systemctl start docker
-> sudo systemctl status docker
-> sudo docker run hello-world
-> sudo docker run -it ubuntu /bin/bash
-> docker images
-> docker ps
-> docker ps -a
-> ls -asl /var/lib/docker
-> docker inspect ubuntu
-> docker start <container-name>
-> docker attach <container-name>
-> sudo docker container prune 
-> sudo docker rmi <container-name>
-> docker commit --change='CMD["python", "-C", "import this"]' <IMAGE ID to use as base> <TAG NAME>
-> docker inspect <IMAGE ID>
-> curl <IP Address>
-> docker stop <container-id...>
-> Detach from container with exiting: Ctrl + p, Ctrl + q

Create image using Dockerfile and running it
-> docker build .
-> docker rmi <IMAGE ID>
-> docker build -t greeting 
-> docker run greeting

Run docker container in background
-> docker run -d greeting

Port Mapping (Container to host)
-> docker run -d -P webapp
-> docker run -d -p 3000:8080 webapp

Networking
-> docker network ls
-> ip addr show
-> arp-scan --interface=eth0 --localnet
-> docker run -d --network=host ubuntu_networking /webapp
-> docker run -it --network=none ubuntu_networking

File storage
-> docker run -d --mount type=volume,src="logs",dst=/logs scratch_volumne
-> tail -F -s1 /var/lib/docker/volumes/logs/_data/myapp
-> cat /var/lib/docker/volumes/logs/_data/myapp | cut -d " " -f2 | sort | uniq
-> docker ps -a
-> docker ps -aq | sort
-> docker run -it --mount type=tmpfs,dst=/logs ubuntu

Tagging
-> docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
-> docker build -t "different_tag_demo" .

Push to docker hub
-> docker login
-> docker tag tag_demo:latest <your_docker_hub_name>/tag_demo:latest
-> docker push <your_docker_hub_name>/tag_demo

Dockerfile
----------

Example 1

FROM scratch
COPY hello /
CMD ["/hello"]

Example 2

FROM scratch
COPY webapp /
EXPOSE 8080
CMD ["/webapp"]

Example 3

FROM ubuntu:16.04

RUN apt update && apt install -y \
    arp scan \
    iputils-ping \
    iproute2

COPY webapp /

CMD ["/bin/bash"]

Resources
---------
-> Cloud Academy
    https://cloudacademy.com/course/introduction-to-docker-2/what-is-docker-1/?context_id=129&context_resource=lp

-> Challenges that docker try to help with
    Deployments, Managing Dependencies, Rollbacks

-> The course assets
https://github.com/cloudacademy/introduction_to_docker

-> Docker installation instructions
https://docs.docker.com/engine/installation/

-> If you want to use Vagrant
https://www.vagrantup.com/docs/index.html

-> The IDE used in the course
https://code.visualstudio.com/

-> A Dockerfile reference
https://docs.docker.com/engine/reference/builder/

-> Tooling for the Go language used in the demos
https://golang.org/

-> support@cloudacademy.com
-> Twitter: @sowhelmed

