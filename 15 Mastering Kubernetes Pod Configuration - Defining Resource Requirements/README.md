Requesting and Limiting Resources for Pods
------------------------------------------

It is a best practice to include some information about the compute resource requirements for a Pod in its specification. The compute resources that Kubernetes includes built-in support for are CPU, measured in cores, and memory, measured in bytes. Kubernetes allows you to include a requested amount (minimum) and a limit (maximum) for each compute resource. Although the compute resource requests and limits are optional, the Kubernetes scheduler can make better decisions about which node to schedule a Pod on if it knows the Pod's resource requirement. The scheduler will only schedule a Pod on a node if the node has enough resources available to satisfy the Pod's total request, which is the sum of the Pod's containers' requests. Limits also reduce resource contention on a node and can make performance more predictable.

1. Load kubectl shell completions for your current shell session:
  ```
  # Load completions
  source <(kubectl completion bash)
  ```

  With completions loaded, you can press tab to autocomplete or list available completions as you enter kubectl commands.

2. Create a Pod manifest for a Pod that will consume a lot of CPU resources:
  ```
  cat << 'EOF' > load.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: load
  spec:
    containers:
    - name: cpu-load
      image: cloudacademydevops/stress
      args:
      - -cpus
      - "2"
  EOF
  ```
  The stress image runs a binary that can consume a varying amount of resources. The args instruct stress to attempt to consume two CPU cores, which is how many cores each node in the cluster has.

3. Create the Pod:
  ```
  kubectl create -f load.yaml
  ```

  The pod simulates a situation where you notice degraded performance in the cluster. When there are many pods and nodes in the cluster, it is difficult to determine which pod or pods are causing the degradation by inspecting each node and pod individually.

4. Display the help document for the top command:
  ```
  kubectl top --help
  ```

  The top command is similar to the native Linux top command for measuring CPU and memory usage of processes. You can monitor at the node or pod level.

5. List the resource consumption of pods:
  ```
  kubectl top pods
  ```

  The load pod is using nearly two full cores (1930milli-cores). Having such an unrestricted pod in your cluster could wreak havoc.

  Note: It can take a minute for the new pod to appear in the list. If you see an error or do not see ithe Pod in the list, re-issue the command every 15 seconds until it appears.



6. Confirm that the Pod is using almost all the CPU for one of the nodes in the cluster:
  ```
  kubectl top nodes
  ```

  Observe one node has a 99% CPU% because each node has only 2 cores.

7. Create a similar Pod specification except with CPU and memory resource limits and requests:
  ```
  cat << 'EOF' > load-limited.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: load-limited
  spec:
    containers:
    - name: cpu-load-limited
      image: cloudacademydevops/stress
      args:
      - -cpus
      - "2"
      resources:
        limits:
          cpu: "0.5" # half a core
          memory: "20Mi" # 20 mebibytes
        requests:
          cpu: "0.25" # quarter of a core
          memory: "10Mi" # 20 mebibytes
  EOF
  ```

  The resources key is added to specify the limits and requests. The Pod will only be scheduled on a Node with 0.25 CPU cores and 10MiB of memory available. It's important to note that the scheduler doesn't consider the actual resource utilization of the node. Rather, it bases its decision upon the sum of container resource requests on the node. For example, if a container requests all the CPU of a node but is actually 0% CPU, the scheduler would treat the node as not having any CPU available.

8. Create the Pod with the resource constraints:
  ```
  kubectl create -f load-limited.yaml
  ```

9. Wait a minute for the new Pod's metrics to be collected, and then display the Pod resource utilization:
  ```
  kubectl top pods
  ```

  Notice the load-limited Pod is using almost half of a CPU core (499milli cores), which is the limit. The request is used for making scheduling decisions but the limit impacts the actual utilization. The Pod was scheduled on the node that isn't running load since load had no CPU available to meet the request. Using requests and limits for CPU and memory can prevent performance issues, and allow the scheduler to make the best use of the cluster's resources.

10. Delete the Pods created in this lab step:
  ```
  kubectl delete -f load.yaml
  kubectl delete -f load-limited.yaml
  ```

The following rules apply when limits or requests are exceeded by Pod containers:

Containers that exceed their memory limits will be terminated and restarted if possible.
Containers that exceed their memory request may be evicted when the node runs out of memory.
Containers that exceed their CPU limits may be allowed to exceed the limit depending on the other Pods on the node. Containers will not be terminated for exceeding CPU limits.
