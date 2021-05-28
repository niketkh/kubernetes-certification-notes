Kubernetes Basics Modules
-------------------------

- Create a Kubernetes cluster
- Deploy an app
- Explore your app
- Expose your app publicly
- Scale your app
- Update your app

Using Minikube to Create a cluster
----------------------------------

A Kubernetes cluster consists of two types of resources:

- The Control Plane coordinates the cluster
- Nodes are the workers that run applications

The Control Plane is responsible for managing the cluster. The Control Plane coordinates all activities in your cluster, such as scheduling applications, maintaining applications' desired state, scaling applications, and rolling out new updates.

A node is a VM or a physical computer that serves as a worker machine in a Kubernetes cluster. Each node has a Kubelet, which is an agent for managing the node and communicating with the Kubernetes control plane. The node should also have tools for handling container operations, such as containerd or Docker. A Kubernetes cluster that handles production traffic should have a minimum of three nodes.

When you deploy applications on Kubernetes, you tell the control plane to start the application containers. The control plane schedules the containers to run on the cluster's nodes. The nodes communicate with the control plane using the Kubernetes API, which the control plane exposes. End users can also use the Kubernetes API directly to interact with the cluster.

- Install minikube which create kubernetes cluster with one node

> minikube version

Start the cluster, by running the minikube start command:
> minikube start

To interact with Kubernetes during this bootcamp we’ll use the command line interface, kubectl.
> kubectl version

Cluster details
> kubectl cluster-info
> kubectl get nodes

Using kubectl to create a deployment
------------------------------------


Viewing Pods and Nodes
----------------------

Pods

A Pod is a Kubernetes abstraction that represents a group of one or more application containers (such as Docker), and some shared resources for those containers. Those resources include:

- Shared storage, as Volumes
- Networking, as a unique cluster IP address
- Information about how to run each container, such as the container image version or specific ports to use

A Pod models an application-specific "logical host" and can contain different application containers which are relatively tightly coupled. For example, a Pod might include both the container with your Node.js app as well as a different container that feeds the data to be published by the Node.js webserver. The containers in a Pod share an IP Address and port space, are always co-located and co-scheduled, and run in a shared context on the same Node.

Pods are the atomic unit on the Kubernetes platform. When we create a Deployment on Kubernetes, that Deployment creates Pods with containers inside them (as opposed to creating containers directly). Each Pod is tied to the Node where it is scheduled, and remains there until termination (according to restart policy) or deletion. In case of a Node failure, identical Pods are scheduled on other available Nodes in the cluster.

Nodes

A Pod always runs on a Node. A Node is a worker machine in Kubernetes and may be either a virtual or a physical machine, depending on the cluster. Each Node is managed by the Master. A Node can have multiple pods, and the Kubernetes master automatically handles scheduling the pods across the Nodes in the cluster. The Master's automatic scheduling takes into account the available resources on each Node.

Every Kubernetes Node runs at least:

Kubelet, a process responsible for communication between the Kubernetes Master and the Node; it manages the Pods and the containers running on a machine.
A container runtime (like Docker) responsible for pulling the container image from a registry, unpacking the container, and running the application.

The most common operations can be done with the following kubectl commands:

kubectl get - list resources
kubectl describe - show detailed information about a resource
kubectl logs - print the logs from a container in a pod
kubectl exec - execute a command on a container in a pod

> kubectl get pods
> kubectl describe pods



Using a Service to expose your app
----------------------------------

Kubernetes Pods are mortal. Pods in fact have a lifecycle. When a worker node dies, the Pods running on the Node are also lost. A ReplicaSet might then dynamically drive the cluster back to desired state via creation of new Pods to keep your application running. As another example, consider an image-processing backend with 3 replicas. Those replicas are exchangeable; the front-end system should not care about backend replicas or even if a Pod is lost and recreated. That said, each Pod in a Kubernetes cluster has a unique IP address, even Pods on the same Node, so there needs to be a way of automatically reconciling changes among Pods so that your applications continue to function.

A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. Services enable a loose coupling between dependent Pods. A Service is defined using YAML (preferred) or JSON, like all Kubernetes objects. The set of Pods targeted by a Service is usually determined by a LabelSelector (see below for why you might want a Service without including selector in the spec).

Although each Pod has a unique IP address, those IPs are not exposed outside the cluster without a Service. Services allow your applications to receive traffic. Services can be exposed in different ways by specifying a type in the ServiceSpec:

ClusterIP (default) - Exposes the Service on an internal IP in the cluster. This type makes the Service only reachable from within the cluster.
NodePort - Exposes the Service on the same port of each selected Node in the cluster using NAT. Makes a Service accessible from outside the cluster using <NodeIP>:<NodePort>. Superset of ClusterIP.
LoadBalancer - Creates an external load balancer in the current cloud (if supported) and assigns a fixed, external IP to the Service. Superset of NodePort.
ExternalName - Maps the Service to the contents of the externalName field (e.g. `foo.bar.example.com`), by returning a CNAME record with its value. No proxying of any kind is set up. This type requires v1.7 or higher of kube-dns, or CoreDNS version 0.0.8 or higher.

Additionally, note that there are some use cases with Services that involve not defining selector in the spec. A Service created without selector will also not create the corresponding Endpoints object. This allows users to manually map a Service to specific endpoints. Another possibility why there may be no selector is you are strictly using type: ExternalName.

A Service routes traffic across a set of Pods. Services are the abstraction that allow pods to die and replicate in Kubernetes without impacting your application. Discovery and routing among dependent Pods (such as the frontend and backend components in an application) is handled by Kubernetes Services.

Services match a set of Pods using labels and selectors, a grouping primitive that allows logical operation on objects in Kubernetes. Labels are key/value pairs attached to objects and can be used in any number of ways:

- Designate objects for development, test, and production
- Embed version tags
- Classify an object using tags

> kubectl get pods
> kubectl get services
> kubectl expose deployment/kubenetes-bootcamp --type="NodePort" --port 8080
> kubectl describe services/kubernetes-bootcamp
> export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
> echo NODE_PORT=$NODE_PORT
> curl $(minikube ip):$NODE_PORT
> kubectl describe deployment
> kubectl get pods -l app=kubernetes-bootcamp
> kubectl get services -l app=kubernetes-bootcamp
> export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
> echo Name of the Pod: $POD_NAME
> kubectl label pod $POD_NAME version=v1
> kubectl describe pods $POD_NAME
> kubectl get pods -l version=v1
> kubectl delete service -l app=kubernetes-bootcamp
> kubectl get services
> curl $(minikube ip):$NODE_PORT
> kubectl exec -ti $POD_NAME -- curl localhost:8080


Running multiple instances of your app
--------------------------------------

Scaling is accomplished by changing the number of replicas in a Deployment
Scaling out a Deployment will ensure new Pods are created and scheduled to Nodes with available resources. Scaling will increase the number of Pods to the new desired state. Scaling to zero is also possible, and it will terminate all Pods of the specified Deployment.

Running multiple instances of an application will require a way to distribute the traffic to all of them. Services have an integrated load-balancer that will distribute network traffic to all Pods of an exposed Deployment. Services will monitor continuously the running Pods using endpoints, to ensure the traffic is sent only to available Pods.

Once you have multiple instances of an Application running, you would be able to do Rolling updates without downtime. We'll cover that in the next module. Now, let's go to the online terminal and scale our application.

> kubectl get deployments
> kubectl get rs
> kubectl scale deployments/kubernetes-bootcamp --replicas=4
> kubectl get deployments
> kubectl get pods -o wide
> kubectl describe deployments/kubernetes-bootcamp
> kubectl describe services/kubernetes-bootcamp
> export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
> echo NODE_PORT=$NODE_PORT
> curl $(minikube ip):$NODE_PORT
> kubectl scale deployments/kubernetes-bootcamp --replicas=2
> kubectl get deployments
> kubectl get pods -o wide

Performing a rolling update
---------------------------

Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.

By default, the maximum number of Pods that can be unavailable during the update and the maximum number of new Pods that can be created, is one. Both options can be configured to either numbers or percentages (of Pods). In Kubernetes, updates are versioned and any Deployment update can be reverted to a previous (stable) version.

Similar to application Scaling, if a Deployment is exposed publicly, the Service will load-balance the traffic only to available Pods during the update. An available Pod is an instance that is available to the users of the application.

Rolling updates allow the following actions:

- Promote an application from one environment to another (via container image updates)
- Rollback to previous versions
- Continuous Integration and Continuous Delivery of applications with zero downtime

> kubectl get deployments
> kubectl get pods
> kubectl describe pods
> kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
> kubectl get pods
> kubectl describe services/kubernetes-bootcamp
> export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
> echo NODE_PORT=$NODE_PORT
> curl $(minikube ip):$NODE_PORT
> kubectl rollout status deployments/kubernetes-bootcamp
> kubectl describe pods


References
----------
- Docs

- Kubernetes Basics
	https://kubernetes.io/docs/tutorials/kubernetes-basics/
 	https://kubernetes.io/docs/concepts/services-networking/connect-applications-service	

- Katakoda (Interactive Learning and Training Platform for Software Engineers)
	https://www.katacoda.com/

- Pod vs Deployment
	
	https://stackoverflow.com/questions/41325087/what-is-the-difference-between-a-pod-and-a-deployment

	You will almost never use an object with the kind pod, because that doesn't make any sense in practice.
	Because you need a deployment object - or other Kubernetes API objects like a replication controller or replicaset - that needs to keep the replicas (pods) alive (that's kind of the point of using kubernetes).

	What you will use in practice for a typical application are:
	- Deployment object (where you will specify your apps container/containers) that will host your app's container with some other specifications.
	- Service object (that is like a grouping object and gives it a so-called virtual IP (cluster IP) for the pods that have a certain label - and those pods are basically the app containers that you deployed with the former deployment object).

	You need to have the service object because the pods from the deployment object can be killed, scaled up and down, and you can't rely on their IP addresses because they will not be persistent.
	So you need an object like a service, that gives those pods a stable IP.
	Just wanted to give you some context around pods, so you know how things work together.

- What's the difference between ClusterIP, NodePort and LoadBalancer service types in Kubernetes
	https://stackoverflow.com/questions/41509439/whats-the-difference-between-clusterip-nodeport-and-loadbalancer-service-types?rq=1