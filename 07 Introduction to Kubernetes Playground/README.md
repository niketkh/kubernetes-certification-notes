Kubernetes is a production-grade container orchestration system that helps you maximize the benefits of using containers. Kubernetes provides you with a toolbox to automate deploying, scaling, and operating containerized applications in production.

Learning Objectives
-------------------

- Deploy single and multiple container applications on Kubernetes
- Use Kubernetes services to structure N-tier applications 
- Manage application deployments with rollouts in Kubernetes
- Ensure container preconditions are met and keep containers healthy
- Learn how to manage configuration, sensitive, and persistent data in Kubernetes

Connecting to Kubernetes Cluster
--------------------------------
You can follow along with the Introduction to Kubernetes Course or to experiment and practice working with a fully functional multi-node Kubernetes cluster in this lab. This lab provides you with a Kubernetes cluster initialized with kubeadm and running on Ubuntu. The cluster is similar to the clusters used in official CNCF Kubernetes certification exams. It has a bastion node for connecting to the Kubernetes cluster nodes that includes a single control-plane node. The infrastructure is deployed on AWS. You can log in to AWS using the lab credentials and Open Environment link when needed.

The following instructions connect to the cluster using a web browser terminal. You can also connect using your preferred SSH client rather than the browser terminal by using the PPK (Windows) or PEM (Mac/Linux) key files in the Credentials section of this lab to connect to the bastion host's IP address which appears below the credentials once it is available.

Note: The Open Environment link to the left takes you to AWS, while the Open Development Env. link takes you to the web terminal.

1. Click Open Development Env. once you see 100% Setup completed
	https://f4ea01fa-bfb7-4eb6-b537-2ca7779a3e0f-code.t.cloudacademylabs.com/

2. At the login prompt enter the following credentials:

login: Enter ca
Password: Press enter (no password)

3. Once you see Open Environment 100% Setup completed 

Copy the Cluster SSH command that appears at the bottom of the lab Credentials panel

> ssh ubuntu@54.186.218.4 -oStrictHostKeyChecking=no

4. In the browser terminal, enter the Cluster SSH command at the prompt

You are now connected to the bastion node in the cluster

5. To ensure the cluster has one available worker node (ROLES set to <none>) before proceeding to use the cluster, enter the following command to watch the cluster come up and press ctrl+c to stop watching

> watch kubectl get nodes

https://282741895229.signin.aws.amazon.com/console?region=us-west-2

Account ID: 282741895229
Username: student
Password: Ca1_B1QQs7y2
Region: US West 2
Bation Public IP: 54.186.218.4
Cluster SSH: ssh ubuntu@54.186.218.4 -oStrictHostKeyChecking=no

Resources
---------

- Kubeadm
	https://kubernetes.io/docs/reference/setup-tools/kubeadm/

- Github
	https://github.com/cloudacademy/intro-to-k8s