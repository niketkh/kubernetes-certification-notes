
Introduction to Kubernetes (Pods)
---------------------------------

1.1-basic_pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: nginx:latest

1.2-port_pod.yaml

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

1.3-labeled_pod.yaml

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

1.4-resources_pod.yaml

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
        cpu: "500m"     # 500m = 500 milliCPUs (1/2 CPU)
      limits:
        memory: "128Mi"
        cpu: "500m"
    ports:
      - containerPort: 80

> kubectl get pods
> cd src
> kubectl create -f 1.1-basic_pod.yaml
> kubectl get pods
> kubectl describe pod mypod | more
> kubectl delete pod mypod
> kubectl create -f 1.2-port_pod.yaml
> kubectl describe pod mypod | more
> curl 192.168.###.###:80 (Replace ###.### with the IP address octets from the describe output)
	# This command will time out (see the next lesson to understand why)
> kubectl delete pod mypod
> kubectl create -f 1.3-labeled_pod.yaml
> kubectl describe pod mypod | more
> kubectl delete pod mypod
> kubectl create -f 1.4-resources_pod.yaml
> kubectl describe pod mypod | more

Note: kubectl will accept the singular or plural form of resource kinds. For example kubectl get pods and kubectl get pod are equivalent.

