Pods are the basic unit of work in Kubernetes.

Learning Objectives
-------------------

Understand how to create Pods and generate Pod YAML manifest files


Lab Environment
---------------

You will use a multi-node Kubernetes cluster to explore pod design considerations in this lab. This lab provides you with a Kubernetes cluster initialized with kubeadm and running on Ubuntu. The cluster is similar to the clusters used in official CNCF Kubernetes certification exams. It has a bastion node for connecting to the Kubernetes cluster nodes that includes a single control-plane node. The infrastructure is deployed on AWS.


- Environment
	https://510937169387.signin.aws.amazon.com/console?region=us-west-2

	Account ID: 510937169387
	Username: student
	Password: Ca1_fQ4d6AWs
	Region: US West 2


- Development Environment
	https://7f3e668a-585d-4cb0-bf90-553321a43ad2-code.t.cloudacademylabs.com/

	At the login prompt enter the following credentials:

	login: Enter ca
	Password: Press enter (no password)

	Run below command to login to cluster
	> ssh ubuntu@52.37.30.65 -oStrictHostKeyChecking=no

	To ensure the cluster has one available worker node (ROLES set to <none>) before proceeding to use the cluster, enter the following command to watch the cluster come up and press ctrl+c to stop watching:

	> watch kubectl get nodes

Reviewing Pod Definition Basics
-------------------------------

Pods are the basic unit of work in Kubernetes. Each Pod is comprised of one or more containers that all share the same network space.

1. Load kubectl shell completions for your current shell session:

> source <(kubectl completion bash)


2. Issue the following command to create a simple Pod manifest file:

cat << 'EOF' > first-pod.yaml
apiVersion: v1            # The API path for the Pod resource
kind: Pod                 # The kind of resource (Pod)
metadata:
  name: first-pod         # Name of the Pod
spec:
  containers:             # List of containers in the Pod
  - image: httpd:2.4.38   # Container image (using a tag to specify version 2.4.38)
    name: first-container # Name of the container
EOF

Read through the comments (following #). The manifest is minimal in the sense that all of the fields are required to create a Pod. For simplicity, some fields that you would usually define for an Apache webserver (httpd), such as container port information, are omitted.

3. Use the explain command to get an explanation of a Pod's spec (press space to page through the output):

> kubectl explain Pod.spec | more

You can always get more details about a resource field by using the explain command. The pattern for the command is

> kubectl explain <Resource_Kind>.<Path_To_Field>

where the <Resource_Kind> is the kind of resource and <Path_To_Field> is the path to the field joined by dots (.). For example, if you wanted to get information about the Pod's container's image field, you could enter

> kubectl explain Pod.spec.containers.image

4. Create the pod by entering:

> kubectl create -f first-pod.yaml

5. Get a summary of the Pod resource by entering:

> kubectl get pod

This command provides a summary of all Pods in the default namespace. Alternatively, you could get a summary for a specific Pod by providing its name as the last argument, e.g. kubectl get pod first-pod.

The READY column indicates how many containers in the Pod are ready. The STATUS column displays Running when all containers have been created and at least one is still running. You can see [this link to learn about the other possible status phases](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase).

6. To get all the fields of a resource, you can use the -o or --output option and specify yaml as the output type as in:

> kubectl get pod first-pod -o yaml | more

Kubernetes adds many more fields compared to the original manifest file. Many of them help Kubernetes manage the resources such as managedFields, resourceVersion, selfLink, uid, and status. However, the fields in the spec are available for you to declare in manifest files to configure your Pods. Take a moment to page through the output and see all of the Pod resource fields.

7. Delete the Pod:

> kubectl delete pod first-pod

You could alternatively specify the -f option to delete the Pod using the file that created it, e.g. kubectl delete -f first-pod.yaml.
