Lab Environment
---------------

1. Click Open Development Env. once you see 100% Setup completed

2. At the login prompt enter the following credentials:

  login: Enter ca
  Password: Press enter (no password)

3. Once you see Open Environment 100% Setup completed

Copy the Cluster SSH command that appears at the bottom of the lab Credentials panel:

> ssh ubuntu@54.214.104.44 -oStrictHostKeyChecking=no

4. In the browser terminal, enter the Cluster SSH command at the prompt

5. To ensure the cluster has one available worker node (ROLES set to <none>) before proceeding to use the cluster, enter the following command to watch the cluster come up and press ctrl+c to stop watching:

> watch kubectl get nodes

Working with Pod Labels, Selectors, and Annotations
---------------------------------------------------

Labels are key-value pairs that are associated with Kubernetes Objects, such as Pods. You can use labels to organize the resources you have in Kubernetes. For example, you may create a label that declared the application tier resources belonged to, such as frontend or backend. Labels do not have to be unique across different resources of a given kind, unlike names and UIDs. Therefore, you can have multiple Pods labeled as frontend.

Label selectors identify a set of Kubernetes Objects using labels. A selector provides conditions for what label should be present (or absent) on Objects and also what values are allowed (or disallowed). For example, a selector could be used to get the set of all Pods that have a tier label or all Pods that have a tier label with a value of frontend.

Labels are attributes that identify resources. Kubernetes also has the concept of annotations, which are non-identifying Object attributes. An example of an annotation is the phone number of a person to call if an issue is discovered with a resource. Just like labels, annotations are also defined as key-value pairs. But you cannot select sets of Objects using annotations. Annotations are often used by Kubernetes client applications (such as kubectl) and Kubernetes extensions.

1. Create a Namespace for the resources you will create in this lab step and change your default kubectl context to use the Namespace:

```
# Create namespace
kubectl create namespace labels
# Set namespace as the default for the current context
kubectl config set-context $(kubectl config current-context) --namespace=labels
```

> kubectl config view

It is a best practice to use namespaces to logically organize your Kubernetes resources. Namespaces identify resources more coarsely than labels and complement labels.

2. Create four Pods declared in the following multi-resource manifest file (pod-labels.yaml):

```
# Write the manifest file
cat << 'EOF' > pod-labels.yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-frontend
  namespace: labels # ceclare namespace in metadata
  labels: # labels mapping in metadata
    color: red
    tier: frontend
  annotations: # Example annotation
    Lab: Kubernetes Pod Design for Application Developers
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: green-frontend
  namespace: labels
  labels:
    color: green
    tier: frontend
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: red-backend
  namespace: labels
  labels:
    color: red
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
---
apiVersion: v1
kind: Pod
metadata:
  name: blue-backend
  namespace: labels
  labels:
    color: blue
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
EOF
# Create the Pods
kubectl create -f pod-labels.yaml
```

Note that multiple resources are separated by --- in YAML manifest files. Each pod is declared in the labels namespace. Each Pod also has a labels mapping that declares a color and a tier label. Values for the color label span red, green, and blue. Values for the tier label span frontend and backend. The first Pod in the file, named red-frontend, also declares an annotations mapping.

3. Use the -L (or --label-columns) kubectl get option to display columns for both labels:

> kubectl get pods -L color,tier

The following instructions use label selectors to select a set of the Pods to display using kubectl. In cases where you don't know what labels are set on Pods, you can use --show-labels option to display all labels for each pod under a single labels column.

4. Use the -l (or --selector) option to select all Pods with a color label:

> kubectl get pods -L color,tier -l color

Specifying only a label key as a selector will select all resources with that label defined. In this case, all Pods have a color label, so they are all selected for output.

5. Select all Pods that do not have a color label:

> kubectl get pods -L color,tier -l '!color'

You can prepend an exclamation mark (!) to select resources without a label defined. The single quotes (') must enclose the selector to prevent the bash shell from interpreting the exclamation mark. You can always enclose your selectors in single quotes to avoid any unexpected consequences of the shell interpreter.

6. Select all Pods that have the color red:

> kubectl get pods -L color,tier -l 'color=red'

You can select a key and value by joining the key and value with an equal sign (=) or two equal signs (==).

7. Select all Pods that have the color red and are not in frontend tier:

> kubectl get pods -L color,tier -l 'color=red,tier!=frontend'

First, note that multiple conditions are joined by commas (,). Second, the != symbol means not equals. For not equals conditions, the given label key must exist. For example, Pods without any tier label would not be selected by the command above.

8. Select all Pods with green or blue color:

> kubectl get pods -L color,tier -l 'color in (blue,green)'

The in condition allows you to specify the allowed values in parentheses. There is also a notin condition that allows you to specify disallowed values.

9. Use the describe command to display the annotation for the red-frontend Pod:

> kubectl describe pod red-frontend | grep Annotations

The describe command is a convenient way to display annotations. You can also set the output option (-o) of get to yaml to view all the fields of resources, including annotations, e.g. kubectl get pod red-frontend -o yaml.

10. Remove the Pod's Lab annotation and verify it no longer exists:

> kubectl annotate pod red-frontend Lab- && kubectl describe pod red-frontend | grep Annotations -A 2

Only annotations related to the cluster’s network plugin remain. The annotate command can be used to add/remove/update annotations. You add a dash (-) after the annotation key to remove the annotation. You can do the same with the kubectl label command when you need to remove a label.

11. Review the Examples in the annotate help:

> kubectl annotate --help

12. Review the Examples in the label help:

> kubectl label --help

The annotate and label commands are analogous. As the differences in the examples highlight, annotations have less restrictions on their values. For example, label values cannot have spaces.

13. Delete the Pods:

> kubectl delete -f pod-labels.yaml
