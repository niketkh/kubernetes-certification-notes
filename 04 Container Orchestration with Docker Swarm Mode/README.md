Docker in Swarm Mode
--------------------

When container applications reach a certain level of complexity or scale, you need to make use of several machines. Container orchestration products and tools allow you to manage multiple container hosts in concert.

- Overlay networks
- Service discovery
- Load balancing
- External access

Points to note
- Containers within a Docker network have access on all ports in same network
- Access is denied between containers that don't share a common network
- Traffic originating inside of a Docker network and not destined for a Docker host is permitted
- Traffic coming into a Docker network is denied by default
- Swarm mode uses the same service discovery system as when not running in swarm mode
- Network can ve overlay spanning multiple hosts, but same internal DNS system is used
- Two services deployed in a swarm, service A, with one replica and service B, with two replicas. When service A makes a request for service B, the VIP of service B is resolved by the DNS server. Using support for IPVS, the request for the VIP address is routes to one of the two nodes runnning service B tasks.


Setting up Swarm
-----------------

Single-node

> docker info | grep Swarm
> docker swarm --help
> docker swarm init
> docker swarm join-token manager
> docker info --format '{{json .Swarm}}'
> docker network ls

Tear down
> docker swarm leave --force

Multi-node

Create and manage machines for docker
> docker-machine create vm1
> docker-machine create vm2
> docker-machine create vm3

> docker-machine ls
> docker-machine ssh vm1
> docker info
> docker swarm init --help
> docker swarm init --advertise-addr=192.168.99.100

Join as worker (vm2, vm3)
> docker-machine ssh vm2
> docker swarm join --token <token-provided-from-init> <IP address>
> docker-machine ssh vm3
> docker swarm join --token <token-provided-from-init> <IP address>

Managing Nodes
--------------
Given swarm with vm1 as manager and vm2, vm3 as workers

- Promoting worker to manager
> docker node --help
> docker node ls
> docker node promote vm2

- Demoting manager to worker
> docker node demote vm2

- Availability (active|pause|drain)
> docker node update --help
> docker node update --availability drain vm1
> docker node ls
> docker node update --availability active vm1

- Labeling
> docker node update --help
> docker node update --label-add zone=1 vm1
> docker node update --label-add zone=2 vm2
> docker node update --label-add zone=3 vm3
> docker node inspect vm3
> docker node inspect -f '{{.Spec.Labels}}' vm3

Managing Services
-----------------
Given swarm with vm1 as manager and vm2, vm3 as workers

- Visualizer 
> docker service --help
> docker service create --constraint=node.role==manager --mode=global --publish mode=host,target=8080,published=8080 --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock --name=viz dockersamples/visualizer
> docker service inspect viz --pretty | more
> docker service ps viz

Use vm1 ip address with port 8080 to access visualiser

> docker node promote vm2
> docker service ls

Use vm1 or vm2 ip address with port 8080 to access visualiser

> docker service create \
	--constraint node.labels.zone!=1 \
	--replicas 2 \
	--placement-pref 'spread=node.labels.zone' \
	-e NODE_NAME='{{.Node.Hostname}}' \
	-p 80:80 \
	--name nodenamer \
	lrakai/nodenamer:1.0.0

> curl localhost
> docker service inspect nodenamer --pretty
> docker service update --image lrakai/nodenamer:1.0.1 nodenamer
> docker service scale nodenamer=6
> docker service update --rollback-parallelism 2 nodenamer
> docker service update --image lrakai/nodenamer:1.0.0 nodenamer
> docker service rollback nodenamer

Working with stacks
-------------------

- Declared in Compose file
- Compose file used in Swarm mode works as a stack
- Version 3+
- docker-stack.yml

> docker service rm viz nodenamer
> docker service ls

Sample template: 

> docker stack --help
> docker stack deploy --help
> docker stack deploy -c docker-stack.yml demo

- change number of replicas and cpu contstraints
> docker stack deploy -c docker-stack.yml demo

Resources
---------

- Docker swarm mode docs
- Swarm github repository

Support
-------
support@cloudacademy.com
Twitter @LoganRakai