Multi-container Pod Patterns
----------------------------
- Sidecar
- Ambassador
- Adapter


Why Pods?
- Pods are smallest unit of deployment in Kubernetes
- Kubernetes needs additional information
- Simplifies using different underlying container runtimes
- Co-locate tightly coupled containers without packaging them as a single image

Sidecar Pattern
- Uses a helper container to assist a primary container
- Commonly used for logging, file syncing, watchers
- Benefits include leaner main container, failure isolation, independent update cycles

Ambassador Pattern
- Ambassador container is a proxy for communicating to and from the primary container
- Commonly used for communicating with databases
- Streamlined development experience, potential to reuse ambassador across languages

Adapter Pattern
- Adapters present a standardized interface across multiple Pods
- Commonly used for normalizing output logs and monitoring data
- Adapts third party software to meet your needs

Refer to https://github.com/cloudacademy/kubernetes-patterns-for-application-developers/tree/master/multi-container-patterns

> kubectl exec -n legacy app -- cat /metrics/raw.txt
> kubectl create -f adapter.yaml
> kubectl exec adapted-pod --container=adapter -- cat /metrics/adapted.json

Networking Topics
-----------------
- Networking Basics
	- Each pod in kubernetes cluster is assigned a unique private IP address
	- All containers in the pod share the same IP address and can communicate with each other using localhost
	- Pods can communicate with each other using the assigned IP address


- Services
	- Services maintain a logical set of pod replicas
	- Usualy these set of pods are identified with labels
	- The service maintains a list of endpoints as pods are added and removed from the set. The service can send requests to any of the pods in the set. Clients of the service now only need to know about the service rather than specific pods.
	- The IP given to a service is called the cluster IP. Cluster IP is the most basic type of service.
	- The cluster IP is only reachable from within the cluster
	- The other types of services allow clients outside of the cluster to connect to the service. The first of those types of services is node port. Node port causes a given port to be opened on every node in the cluster. The cluster IP is still given to the service. Any requests to the node port of any node are routed to the cluster IP. 
	- The next type of service that allows external access is load balancer. The load balancer type exposes the service externally through a cloud provider's load balancer. A load balancer type also creates a cluster IP and a node port for the service. Requests to the load balancer are sent to the node port and routed to the cluster IP. Different features of cloud provider load balancers, such as connection draining and health checks, are configured using annotations on a load balancer. The final type of service is the external name and it is different in that it is enabled by DNS, not proxying.

- Network Policy
	- Network policies are similar to simple firewalls or security groups that control access to virtual machines running in a cloud. Network policies are namespace resources meaning that you can configure network policies independently for each Kubernetes namespace
	- The container network plugin running in your cluster must support network policies to get any of their benefits. Otherwise, you will create network policies and there won't be anything to enforce them. In the worse case, you might think that you have secured access to an application but the pods are actually still open to request from anywhere.

	> kubectl get pods -n kube-system
	> kubectl get pods -n network-policy -L app,region
	> kubectl logs -n network-policy --tail=1 -f client-1
	> kubectl logs -n network-policy --tail=1 -f client-2
	> kubectl get -n network-policy networkpolicies
	> kubectl logs -n network-polcicy --tail=0 -f client-2
	> kubectl logs -n network-polcicy --tail=0 -f client-1
	> kubectl delete networkpolicies -n network-policy allow-us-east
	> kubectl logs -n network-polcicy --tail=0 -f client-1
	> kubectl exec -n network-policy server -it ping 192.168.134.72
	> kubectl exec -n network-policy server -it ping 192.168.233.198

	- The rules apply to new connections. In the case of a client sending a request to the server, the connection is already in place and the response is sent back through the same existing connection. There is no new connection so the network policy can't block the egress

https://github.com/lrakai/kubernetes-patterns-for-application-developers/blob/master/network-policy/0-pods.yaml
https://github.com/lrakai/kubernetes-patterns-for-application-developers/blob/master/network-policy/1-allow-us-east.yaml
https://github.com/lrakai/kubernetes-patterns-for-application-developers/blob/master/network-policy/2-block-one-ip.yaml

Service Accounts
----------------
- Provide an identity to Pods in the cluster
- Stored in, and managed by the cluster

Service accounts are namespace resources meaning they can only be used within one Kubernetes namespace and name of service accounts must be unique within the namespace but not across different namespaces. One of the main reasons for using service accounts is to utilize Role-Based Access Control or RBAC is securely mechanism built into Kubernetes. RBAC largely to define roles and associated permissions to access Kubernetes APIs and resources.

A Kubernetes administrator would usually define their roles since it requires knowledge of Kubernetes APIs and small mistakes could compromise the entire cluster. The actual mechanics of a pod authenticating itself involve using an authentication token that is automatically mounted into the pods file system. That token can be used as an authentication header for request center to Kubernetes APIs server. Every namespace has a default service account that is named default. Every pod that doesn't specify service account automatically uses a default service account. The default service account doesn't have any additional permissions than an authenticated user, making it secured by default. Those permissions are mainly limited to discovering what API the cluster provides but does not grab permission to use them, thereby denying access to any cluster state. Let's go see how service accounts can be configured. 

> kubectl get -n kube-system serviceaccounts | more
> kubectl describe -n kube-system clusterrole system:coredns
> kubectl get -n kube-system pod coredns-<hash> -o yaml | less

Image pull secrets come into play when you're using a private container registry for your container images. 

The way that you authenticate with the container registry so that you can pull an image into Kubernetes is by using image pull secrets. The secrets may be a docker registry server address user name and a password, for example, you can specify image pull secrets in pod specs directly but service accounts can also reference image pull secrets. Service accounts can provide both an authentication token and image pull secrets. It creates a nice separation of responsibilities when uses a service account for managing image pull secrets rather than hard coring your reference into pods specs.

Leveraging kubectl
------------------

> kubectl | more
> kubectl completion --help
> source <(kubectl completion bash)
> echo "source <(kubectl completion bash)" >> ~/.bash_profile
> kubectl (tab twice)
> kubectl get (tab twice)
> kubectl get nodes
> kubectl api-resources
> kubectl get csr
> kubectl get pods --all-namespaces
> kubectl get pods --all-namespaces --show-labels
> kubectl get pods --all-namespaces -L k8s-app -l k8s-app
> kubectl get pods --all-namespaces -L k8s-app -l k8s-app=kube-proxy
> kubectl get pods --all-namespaces -L k8s-app -l k8s-app!=kube-proxy
> kubectl get pods --all-namespaces -L k8s-app -l k8s-app!=kube-proxy,k8s-app
> kubectl get pods -n kube-system --sort-by='{.metadata.creationTimestamp}'
> kubectl get pods -n kube-system kube-proxy-<hash> --output=yaml
> kubectl get pods -n kube-system kube-proxy-<hash> -o yaml
> kubectl get pods -n kube-system --sort-by='{.status.podIP}'
> kubectl get pods -n kube-system kube-proxy-<hash> -o wide
> kubectl get pods -n kube-system --sort-by='{.status.podIP}' -o jsonpath='{.items[*].status.podIP}'
> kubectl get pods -n kube-system --sort-by='{.status.podIP}' -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'
> kubectl create -h | less
> kubectl create namespace tips -o yaml --dry-run
> kubectl create namespace tips -o yaml --dry-run > tips/1-namespace.yaml
> kubectl run nginx --image=nginx --port=80 --replicas=2 --expose --dry-run -o yaml
> kubectl run nginx --image=nginx --port=80 --replicas=2 --expose --dry-run -o yaml > tips/2-deployment.yaml
> vimtutor
> kubectl create -f tips
> kubectl delete -f tips
> kubectl get pod -n kube-system kube-proxy-<random> -o yaml | wc -l
> kubectl get pod -n kube-system kube-proxy-<random> -o yaml --export | wc -l
> kubectl explain -h | more
> kubectl explain pod | more
> kubectl explain pod.spec.containers.resources | more
> kubectl explain pod.spec.containers.resources --recursive | more

References
----------