Kubernetes
----------

- Open source, container orchestration tool designed to automate, deploying, scaling, and operating containerized applications
- Born out of Google's experience running production workloads at scale
- Allows organizations to increase their velocity by releasing and recovering faster

Feature
- Automated deployment rollout and rollback
- Seamless horizontal scaling
- Secret management
- Service discovery and load balancing
- Linux and Windows container support
- Simple log collection
- Stateful application support
- Persistent volume management
- CPU and memory quotas
- Batch job processing
- Role based access control

Alternatives
- DCOS
- Amazon ECS
- Docker swarm mode

Kubernetes commands 
Kubernetes Web dashboard



Kubernetes commands
-------------------

Create resources (Pods, Services, etc)
> kubectl create

Delete resources (Pods, Services, etc)
> kubectl delete

Get list of resources of given type
> kubectl get
> kubectl get pods

Describe detailed info about resource(s)
> kubectl describe
> kubectl describe pod server

Container Logs
> kubectl logs


Pods
----

- Basic building block in Kubernetes
- One or more containers in a Pod
- Pod containers all share a container network
- One IP address per pod
- Declaration includes container image, container ports, container restart policy, resource limits

- Basic pod yml (1.1-basic_pod.yaml)

apiVersion: v1
kind: Pod
metadata: 
	name: mypod
spec: 
	containers: 
	- name: mycontainer
	  image: nginx:latest

> kubectl create -f 1.1-basic-pod.yaml
> kubectl get pods
> kubectl describe pod mypod | more

- Basic pod with port yml (1.2-port_pod.yaml)

apiVersion: v1
kind: Pod
metadata: 
	name: mypod
spec: 
	containers: 
	- name: mycontainer
	  image: nginx:latest
	  ports: 
	  	- containerPort: 80

> kubectl delete pod mypod
> kubectl create -f 1.2-port_pod.yaml
> kubectl describe pod mypod | more

- Basic pod with port and label yml (1.3-labeled_pod.yaml)

apiVersion: v1
kind: Pod
metadata: 
	name: mypod
	labels: 
		app: webserver
spec: 
	containers: 
	- name: mycontainer
	  image: nginx:latest
	  ports: 
	  	- containerPort: 80

- Add resource requests (1.4-resources_pod.yaml)

apiVersion: v1
kind: Pod
metadata: 
	name: mypod
	labels: 
		app: webserver
spec: 
	containers: 
	- name: mycontainer
	  image: nginx:latest
	  resources:
	  	requests:
	  		memory: "128Mi" # 128Mi = 128 mebibytes
	  		cpu: "500m" 	# 500m = 500 milliCPUs (1/2 CPU)
	  	limits:
	  		memory: "128Mi"
	  		cpu: "500m"
	  ports: 
	  	- containerPort: 80

> kubectl create -f 1.4-resources_pod.yaml
> kubectl describe pod mypod | more
 
Services
--------

- 2.1-web_service.yaml

apiVersion: v1
kind: Service
metadata: 
	labels: 
		app: webserver
	name: webserver
spec: 
	ports: 
	- port: 80
	selector:
		app: webserver
	type: NodePort

> kubectl create -f 2.1-web_service.yaml
> kubectl get services
> kubectl describe services webserver
> kubectl describe nodes | grep -i  address -A 1
> curl 192.168.62.2:32337

Multi-container Pods and Namespaces
-----------------------------------

- Namespaces separate resources according to users, environments, or applications
- Role based access control (RBAC) to secure access per Namespace

- 3.1-namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
	name: microservice
	labels: 
		app: counter

> kubectl create -f 3.1-namespace.yaml

- 3.2-multi-container.yaml

apiVersion: v1
kind: Pod
metadata: 
	name: app
spec: 
	containers:
	- name: redis
	  image: redis:latest
	  imagePullPolicy: IfNotPresent
	  ports:
	  	- containerPort: 6379

	- name: server
	  image: lrakai/microservices:server-v1
	  ports: 
	  	- containerPort: 8080
	  env:
	  	- name: REDIS_URL
	  	  value: resis://localhost:6379

	- name: counter
	  image: lrakai/microservices:counter-v1
	  env:
	  	- name: API_URL
	  	  value: http://localhost:8080

	- name: poller
	  image: lrakai/microservices:poller-v1
	  env: 
	  	- name: API_URL
	  	  value: http://localhost:8080

> kubectl create -f 3.2-multi-container.yaml -n microservice
> kubectl get -n microservice pod app
> kubectl describe -n microservice pod app
> kubectl logs -n microservice app counter --tail 10
> kubectl logs -n microservice app poller -f


Service Discovery (Multi-Tier)
-----------------

- Environment Variables
	- Services address automatically injected in containers
	- Environment variables follow naming conventions based on service name

- DNS
	- DNS records automatically created in cluster's DNS
	- Containers automatically configured to query cluster DNS


- 4.1-namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
	name: service-discovery
	labels: 
		app: microservices

- 4.2-data_tier.yaml

apiVersion: v1
kind: Service
metadata: 
	name: data-tier
	labels: 
		app: microservices
spec: 
	ports:
	- port: 6379
	  protocol: TCP # default
	  name: redis 	# optional when only 1 port
	selector: 
		tier: data
	type: ClusterIP # default
---
apiVersion: v1
kind: Pod
metadata: 
	name: data-tier
	labels: 
		app: microservices
		tier: data
spec: 
	containers: 
		- name: redis
		  image: redis:latest
		  imagePullPolicy: IfNotPresent
		  ports: 
		  	- containerPort: 6379


> kubectl create -f 4.2-data_tier.yaml -n service-discovery
> kubectl get pod -n service-discovery data-tier

- 4.3-app_tier.yaml

appVersion: v1
kind: Service
metadata: 
	name: app-tier
	labels: 
		app: microservices
spec: 
	ports: 
	- port: 8080
	selector:
		tier: app
---
apiVersion: v1
kind: Pod
metadata: 
	name: app-tier
	labels: 
		app: microservices
		tier: app
	spec: 
		containers:
			- name: server
			  image: lrakai/microservices:server-v1
			  ports:
			  	- containerPort: 8080
			  env:
			    - name: REDIS_URL
			      # Environment variable service discovery
			      # Naming pattern: 
			      #   IP address: <all_caps_service>_SERVICE_HOST
			      #   Port: <all_caps_service_name>_SERVICE_PORT
			      #   Named Port: <all_caps_service_name>_SERVICE_PORT_<all_caps_port_name>
			      value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_TIER_SERVICE_PORT_REDIS)
			      # In multi-container example value was
			      # value: redis://localhost:6379

> kubectl create -f 4.3-app_tier.yaml -n service-discovery

- 4.4-support_tier.yaml

appVersion: v1
kind: Pod
metadata: 
	name: support-tier
	labels:
		app: mciroservices
		tier: support
	spec:
		containers:
		- name: counter
		  image: lrakai/microservices:counter-v1
		  env: 
		  	- name: API_URL
		  	  # DNS for service discovery
		  	  # Naming pattern: 
		  	  #   IP address: <service_name>.<service_namespace>
		  	  #   Port: needs to be extracted from SRV DNS record
		  	  value: http://app-tier.service-discovery:8080

		- name: poller
		  image: lrakai/microservices:poller-v1
		  env: 
		  	- name: API_URL
		  	# omit namespace to only search in the same namespace
		  	value: http://app-tier:$(APP_TIER_SERVICE_PORT)

> kubectl create -f 4.4-support_tier.yaml -n service-dsicovery
> kubectl get pods -n service-discovery
> kubectl logs -n service-discovery support-tier poller -f

Deployments
-----------
- Represents multiple replicas of s Pod
- Describe a desired state that Kubernetes needs to achieve
- Deployment Controller master component converges actual state to the desired state

- 5.1-namespace.yaml
// deployments namespace

- 5.2-data-tier.yaml

apiVersion: v1
kind: Service
metadata: 
	name: data-tier
	labels: 
		app: microservices
spec: 
	ports:
	- port: 6379
	  protocol: TCP # default
	  name: redis 	# optional when only 1 port
	selector: 
		tier: data
	type: ClusterIP # default
---	
apiVersion: apps/v1 # apps API group
kind: Deployment
metadata: 
	name: data-tier
	labels: 
		app: microservices
		tier: data
spec: 
	replicas: 1
	selector: 
		matchLabels:
			tier: data
	template:
		metadata: 
			labels: 
				app: microservices
				tier: data
		spec: # Pod spec
			containers:
			- name: redis
			  image: redis:latest
			  imagePullPolicy: IfNotPresent
			  ports: 
			  	- containerPort: 6379

> diff -y 4.2-data_tier.yaml 5.2-data_tier.yaml

Similar changes to app-tier and support-tier to use deployment and replicas

> kubectl create -n deployments -f 5.2-data_tier.yaml -f 5.3-app_tier.yaml -f 5.4-support_tier.yaml
> kubectl get -n deployments deployments
> kubectl -n deployments get pods
> kubectl scale -n deployments deployment support-tier --replicas=5
> kubectl -n deployments get pods
> kubectl delete -n deployments pods support-tier-<hash>
> kubectl scale -n deployments deployment app-tier --replicas=5
> kubectl -n deployments get pods
> kubectl describe -n deployments service app-tier

Autoscaling
-----------

- Scale automatically based on CPU utilization or custom metrics
- Set target CPU along with min and max replicas
- Target CPU is expressed as a percentage of Pod's CPU request

> kubectl apply -f metrics-server/
> kubectl top pods -n deployments
> kubectl create -f 6.1-app_tier_cpu_request.yaml -n deployments
> kubectl apply -f 6.1-app_tier_cpu_request.yaml -n deployments
> kubectl get -n deployments deployments app-tier
> kubectl autoscale deployment app-tier --max=5 --min=1 --cpu-percent=70
> kubectl create -f 6.2-autoscale.yaml -n deployments
> watch -n 1 kubectl get -n deployments deployment
> kubectl api-resources
> kubectl describe -n deployments hpa
> kubectl get -n deployments hpa
> kubectl edit -n deployments hpa
> watch -n1 kubectl get -n deployments deployment

Rolling updates and rollbacks
-----------------------------

> kubectl delete -n deployments hpa app-tier 
> kubectl edit -n deployments deployment app-tier
> watch get -n deployments deployment app-tier
> kubectl edit -n deployments deployment app-tier

// change container name and do a rollout
> kubectl rollout -n deployments status deployment app-tier
> tmux
> kubectl rollout -n deployments pause deployment app-tier
> kubectl rollout -n deployments resume deployment app-tier
> kubectl rollout -n deployments undo deployment app-tier
> kubectl scale -n deployments deployment app-tier --replicas=1

Probes
------

Readiness Probe
- Sometimes referred to as health checks
- Used to check when a Pod is ready to serve traffic/handle request
- Useful after startup to check external dependencies
- Readiness Prbes set the Pod's ready condition. Services only send traffic to ready Pods

Liveness Probe
- Detect when a Pod enters a broken state
- Kubernetes will restart the Pod for you
- Declared in the same way as readiness probes

- Probes can be declared in a Pod's container
- All container probes must pass for the Pod to pass
- Probe actions be a command that runs in the container, an HTTP GET request, or opening a TCP socket
- By default, probes check the container every 10 seconds


- 7.1-namespace.yaml # create probes namespace
- 7.2-data-tier.yaml

...
livenessProbe:
	tcpSocket: 
		port: redis # named port
	initalDelaySeconds: 15
readinessProbe:
	exec:
		command:
		- redis-cli
		- ping
	initalDelaySeconds: 5
...

> kubectl create -f 7.2-data-tier.yaml -n probes
> kubectl get deployments -n probes -w

- 7.3-app_tier.yaml 

...
	- name: DEBUG
  		value: express:*
livenessProbe: 
	httpGet:
		path: /probe/liveness
		port: server
	initalDelaySeconds: 5
readinessProbe:
	httpGet:
		path: /probe/readiness
		port: server
	initalDelaySeconds: 3
...

> kubectl create -f 7.3-app-tier.yaml -n probes
> kubectl get deployments -n probes app-tier -w
> kubectl get -n probes pods
> kubectl logs -n probes app-tier-<hash> | cut -d' ' -f5,8-11

Init Containers
---------------

- Sometimes you need to wait for service, downloads, dynamic, or decisions before starting Pod's containers
- Prefer to separate initalization wait logic from container image
- Initalization is tightly coupled to the main application (belongs in the Pod)
- Init containers allow you to run initialization tasks before starting main container(s)

- 9.1-app_tier.yaml
...
initContainers: 
	- name: await-redis
	  image: lrakai/microservices:server-v1
	  env: 
	  - name: REDIS_URL
	    value: redis://$(DATA_TIER_SERVICE_HOST):$(DATA_T...
	  command:
	  	- npm
	  	- run-script
	  	- await-redis
...

> kubectl apply -f 8.1-app_tier.yaml -n probes
> kubectl get pods -n probes
> kubectl describe pod app-tier-<hash> -n probes
> kubectl logs -n probes app-tier-<hash> -c await-redis

Volumes
-------

Motivation behind volumes
- Sometimes useful to share data between containers in a Pod
- Lifetime of container file systems is limited to container's lifetime
- Can lead to unexpected consequences if a container restarts

How to handle non-ephemeral data
- Volumes
- Persistent Volumes
- Used by mounting a director in one or more containers in a Pod
- Pods can use multiple volume and persistent volumes
- Difference between volumes and persistent volumes is how their lifetime is managed

Volumes
- Volumes are tied to a Pod and their lifecycle
- Share data between containers and tolerate container restarts
- Use for non-durable storage that is deleted with the Pod
- Default Volume type is emptyDir
- Data is lost if Pod is rescheduled on a different node

Persistent Volumes
- Independent of Pod's lifetime
- Pods claim Persistent Volumes to use throughout their lifetime
- Can be mounted by multiple Pods on different Nodes if underlying storage supports it 

Persistent Volumes Claims
- Describe a Pod's request for Persistent Volume storage
- Includes how much storage, type of storage, and access mode
- Access mode can be read write once, read only many, read write many
- PVC stays pending if no PV can satisfy it and dynamic provisioning is not enabled

Storage Volume Types
- Wide variety of volume types to choose from 
- Use persistent volumes for more durable storage types
- Supported durable storage types include GCE Persistent Disks, Azure Disks, Amazon EBS, NFS, and iSCSI

> kubectl -n deployments logs support-tier-<hash> poller --tail=1
> kubectl get pods -n deployments
> kubectl exec -n deployments data-tier-<hash> -it -- /bin/bash
> kill 1
> watch -n5 kubectl -n deployments logs support-tier-<hash> poller --tail=1

- 9.1-namespace.yaml
# Create a volumes namespace

> kubectl create -f 9.1-namespace.yaml
> diff -y 7.2-data_tier.yaml 9.2-pv_data_tier.yaml

...
apiVersion: v1
kind: PersistentVolume
metadata: 
	name: data-tier-volume
spec: 
	capacity: 
		storage: 1Gi # 1 Gibibyte
	accessMode: 
		- ReadWriteOnce
	awsElasticBlockStore: 
		volumeID: INSERT_VOLUME_ID # replace with actual ID
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
	name: data-tier-volume-claim
spec: 
	accessModes: 
		- ReadWriteOnce
	resources: 
		requests:
			stirage: 128Mi # 128 mebibytes
...
# Within spec for redis container
...
	volumeMounts:
		- mountPath: /data
		  name: data-tier-volume
volumes:
- name: data-tier-volume
  persistentVolumeClaim:
  	claimName: data-tier-volume-claim

> aws ec2 describe-volumes --region=us-west --filters="Name=tag:Type,Values=PV" --query="Volumes[0].VolumeId" --output=text
> sed -i "s/INSERT_VOLUME_ID/$vol_id/" 9.2-pv_data_tier.yaml
> cat 9.2-pv_data_tier.yaml | greo $vol_id -C5
> kubcectl create -n volmes -f 9.2-pv_data_tier.yaml -f 9.3-app_tier.yaml 9.4-support_tier.yaml
> kubectl describe -n volumes pvc
> kubectl get pods -n volumes
> kubectl describe -n volumes pod data-tier-<hash>
> watch -n5 kubectl logs -n volumes support-tier-<hash> poller  --tail=1
> kubectl delete -n volumes deployments data-tier
> kubectl create -f 9.2-pv_data_tier.yaml -n volumes

ConfigMaps and Secrets
----------------------

- Until now all container configuration has been in Pod spec
- This makes it less portable than it could be
- If sensitive information such as API keys and passwords is involved it presents a security issue

- 10.2-data_tier_config.yaml
apiVersion: v1
kind: ConfigMap
metadata: 
	name: redis-config
data:
	config: | #YAML for multi-line string
	# Redis config file
	tcp-keepalive 240
	maxmemory 1mb

- 10.3-data_tier.yaml
...
	command:
		- redis-server
		- /etc/redis/redis.conf
	volumeMounts: 
		- mountPath: /etc/redis
		  name: config
volumes:
- name: config
  configMap: 
  	name: redis-config
  	items: 
  		- key: config
  		  path: redis.conf		  

> kubectl create -n config -f 10.2-data_tier_config.yaml -f 10.3-data_tier.yaml
> kubectl -n config get pods
> kubectl exec -n config data-tier-<hash> =it -- /bin/bash
> cat /etc/redis/redis.conf
> redis-cli CONFIG GET tcp-keepalive

- 10.4-app_tier_secret.yaml
apiVersion: v1
kind: Secret
metadata: 
	name: app-tier-secret
stringData: # unencoded data
	api-key: sdhkjhffhdlfjk
	decoded: hello
data: # for base64 encoded data
	encoded: aGVsbG8= #hello in base-64
# api-key secret (only) is equivalent to 
# kubectl create secret geberic app-tier-secret --from-literal=api-key=sdhkjhffhdlfjk

> kubectl create -f 10.4-app_tier_secret.yaml -n config
> kubectl describe -n config secrets app-tier-secret
> kubectl edit -n config secrets app-tier-secret

- 10.5-app_tier.yaml

...
- name: API_KEY
  valueFrom:
  	secretKeyRef:
  		name: app-tier-secret
  		key: api-key
...

> kubectl create -f 10.5-app_tier.yaml -n config
> kubectl get pods -n config
> kubectl exec -n config app-tier-<hash> -- env

Kubernetes Ecosystem
--------------------

- Helm
- Kustomize
- Prometheus
- Kubeflow
- Knative

References
----------
- Github Link
	github.com/cloudacademy/intro-to-k8s

- Metrics server
	github.com/kubernetes-incubator/metrics-server

- Kubernetes docs
	kubernetes.io/docs

Contacts
--------
- Jonathan Lewey
- support@cloudacademy.com