---
title: Simulation Testing
---

# Simulation Testing with KWOK

{{< hint "info" >}}

This document walks you through the technical outcome of using KWOK for simulation testing.

{{< /hint >}}

This article describes the technical outcome of using KWOK for simulation testing in your environment.
It goes over the key simulation features of using KWOK, and then provides a step-by-step guide to creating
various scheduling scenarios using KWOK.

## Key Simulation Features of KWOK

1. **Lifecycle configuration:** The pod, node, and node lease can be simulated.

    - **Pod:** Status information like Pending, ContainerCreating, Running, and Terminating can be simulated.
    - **Node:** Status information like NodeReady, and Ready states can be simulated.

2. **Kubelet server**

    - **Metrics:** Using the [Kubernetes Metric Server](https://kwok.sigs.k8s.io/docs/examples/metrics-server/), or [Prometheus](https://kwok.sigs.k8s.io/docs/examples/prometheus/), metrics can be simulated in KWOK.
    - **Exec:** Commands can be simulated to be [executed](https://kwok.sigs.k8s.io/docs/user/exec-configuration/) inside a container running on a single fake pod or multiple pods.
    - **Log:** Container [logs](https://kwok.sigs.k8s.io/docs/user/logs-configuration/) can be simulated in a single pod or multiple pods.
    - **Attach:** The input/output streams from the main process running in a container can be [attached](https://kwok.sigs.k8s.io/docs/user/attach-configuration/#attach-configuration) to KWOK.
    This can be simulated in a single pod or multiple pods.
    - **PortForward:** [Port forwarding](https://kwok.sigs.k8s.io/docs/user/port-forward-configuration/#clusterportforward), which establishes a direct connection to a container port, can be simulated
    for a single pod or multiple pods.

## Technical Outcome

### Scheduler Simulation

KWOK can be used to create fake nodes and pods in a simulated cluster.
The cluster can be configured with scheduling policies that meet your scheduler's requirements.
These scenarios will be used to describe this.

- Scenario 1: Scheduling pods with resource requests and limits
- Scenario 2: Scheduling a pod to a particular node with node-affinity
- Scenario 3: Scheduling pods with taints and tolerations
- Scenario 4: Scheduling pods with a limit range
- Scenario 5: Scheduling pods using pod priority and preemption
- Scenario 6: Scheduling pods using pod topology spread constraints

#### Scenario 1: Scheduling pods with resource requests and limits

This image shows you what you should expect when testing this scenario.
You can follow the step-by-step guide after seeing this.

<img width="700px" src="simulation/scenario-1/README.svg">

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

- This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy nodes

Below are the node resource specifications:

- Nodes: 2 worker nodes in the cluster
  - Node 1: 4 CPUs
  - Node 2: 2 CPUs

{{< expand "simulation/scenario-1/node.yaml" >}}

{{< code-sample file="simulation/scenario-1/node.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f node.yaml
```

Pod resource usage specifications

- Pod 1: Requests 2.7 CPU. Limits 3 CPU.
- Pod 2: Requests 1.2 CPU. Limits 1.5 CPU.

Scheduling process

Step 1: Deploy Pod 1 and 2

{{< expand "simulation/scenario-1/pod-1.yaml" >}}

{{< code-sample file="simulation/scenario-1/pod-1.yaml" >}}

{{< /expand >}}

{{< expand "simulation/scenario-1/pod-2.yaml" >}}

{{< code-sample file="simulation/scenario-1/pod-2.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f pod-1.yaml
kubectl apply -f pod-2.yaml
```

Step 2: View the node the pod is scheduled to

```bash
kubectl get pod -o wide

NAME    READY   STATUS    RESTARTS   AGE   IP         NODE     NOMINATED NODE   READINESS GATES
pod-1   1/1     Running   0          5s    10.0.0.1   node-1   <none>           <none>
pod-2   1/1     Running   0          5s    10.0.0.2   node-2   <none>           <none>
```

- Pod 1 is scheduled to Node 1.
- Pod 2 is scheduled to Node 2.

Step 3: View node resource usage

```bash
kubectl describe node node-1 | awk '/Allocated resources:/,/ephemeral-storage/'
kubectl describe node node-2 | awk '/Allocated resources:/,/ephemeral-storage/'
```

This will provide detailed information about the node's resource capacity and usage.

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling scenario based on [resource requests and limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) policy.

#### Scenario 2: Scheduling a pod to a particular node with node-affinity

This image shows you what you should expect when testing this scenario.
You can follow the step-by-step guide after seeing this.

<img width="700px" src="simulation/scenario-2/README.svg">

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

- This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy nodes

Below are the node resource specifications:

- Nodes: 2 worker nodes in the cluster

```bash
kubectl apply -f node.yaml
```

{{< expand "simulation/scenario-2/node.yaml" >}}

{{< code-sample file="simulation/scenario-2/node.yaml" >}}

{{< /expand >}}

Label node-1

```bash
kubectl label node node-1 region=us-west-2
```

Pod to be scheduled

The pod has a node affinity configured with a key value of “region=eu-west-2”. This matches the label on node-1.

Scheduling process

Step 1: Deploy pod

{{< expand "simulation/scenario-2/pod.yaml" >}}

{{< code-sample file="simulation/scenario-2/pod.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f pod.yaml
```

Step 2: View the node the pod is scheduled to.

```bash
kubectl get pod -o wide
```

- Pod 1 is scheduled to Node 1.

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling 
scenario based on [node affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/) policy.

#### Scenario 3: Scheduling Pods with taints and tolerations

A taint and toleration can be defined in KWOK. It’s handy in these situations:

1. You have KWOK installed in your real cluster (KIND, K3D, etc.), and you want some or all of your pods to be scheduled to the KWOK nodes to test for scalability.
2. You want to simulate taint and toleration use cases within a KWOK cluster.

Let's look at **point 2** for now.

This image shows you what you should expect when testing this scenario.
You can follow the step-by-step guide after seeing this.

<img width="700px" src="simulation/scenario-3/README.svg">

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

- This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy node

- Node: 1 worker node in the cluster

{{< expand "simulation/scenario-3/node.yaml" >}}

{{< code-sample file="simulation/scenario-3/node.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f node.yaml
```

Taint node

```bash
kubectl taint nodes node kwok.x-k8s.io/node=fake:NoSchedule
```

Deploy a pod without toleration and observe

{{< expand "simulation/scenario-3/no-toleration-pod.yaml" >}}

{{< code-sample file="simulation/scenario-3/no-toleration-pod.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f no-toleration-pod.yaml
kubectl get pod

NAME                READY   STATUS    RESTARTS   AGE
no-toleration-pod   0/1     Pending   0          4s
```

The pod is stuck in a pending state.

Deploy a pod with toleration and observe

{{< expand "simulation/scenario-3/with-toleration-pod.yaml" >}}

{{< code-sample file="simulation/scenario-3/with-toleration-pod.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f with-toleration-pod.yaml

kubectl get pod

NAME                    READY   STATUS    RESTARTS   AGE
no-toleration-pod       0/1     Pending   0          20s
with-toleration-pod     1/1     Running   0          2s
```

Only the pod with toleration is in a running state.

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling
scenario based on [taints and tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) policy.

#### Scenario 4: Scheduling pods with a limit range

A limit range schedule policy can be used in a KWOK cluster.

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy node

- Node: 1 worker node in the cluster

{{< expand "simulation/scenario-4/node.yaml" >}}

{{< code-sample file="simulation/scenario-4/node.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f node.yaml
```

Create a resource limit

{{< expand "simulation/scenario-4/limit-range.yaml" >}}

{{< code-sample file="simulation/scenario-4/limit-range.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f limit-range.yaml
```

Confirm the limit has the required values

```bash
kubectl describe limitranges cpu-resource-constraint

Name:       cpu-resource-constraint
Namespace:  default
Type        Resource  Min   Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---   ---  ---------------  -------------  -----------------------
Container   cpu       100m  1    500m             500m           -
```

Deploy a pod above the resource limit

- Pod specification: 0.7 CPU

{{< expand "simulation/scenario-4/pod-beyond-limit.yaml" >}}

{{< code-sample file="simulation/scenario-4/pod-beyond-limit.yaml" >}}

{{< /expand >}}

```bash
kubectl create -f pod-beyond-limit.yaml
```

Notice the error `Invalid value: "700m": must be less than or equal to cpu limit of 500m`

Deploy a pod within the resource limit

- Pod specification: 0.4 CPU

{{< expand "simulation/scenario-4/pod-within-limit.yaml" >}}

{{< code-sample file="simulation/scenario-4/pod-within-limit.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f pod-within-limit.yaml
```

Confirm that the pod is running

```bash
kubectl get pod

NAME                 READY   STATUS    RESTARTS   AGE
pod-with-new-limit   1/1     Running   0          10s
```

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling
scenario based on setting a [limit range](https://kubernetes.io/docs/concepts/policy/limit-range/) policy.

#### Scenario 5: Scheduling pods using pod priority and preemption

Pod priority and preemption scheduling policies can be applied in a KWOK cluster.
For this particular scenario, the cluster will be limited to a particular resource range, then
pod priority and preemption policy policies will be used to evict low-priority pods.

This image shows you what you should expect when testing this scenario.
You can follow the step-by-step guide after seeing this.

<img width="700px" src="simulation/scenario-5/README.svg">

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy node

Below is the node resource specification:

- Node: 1 worker node in the cluster
  - Node 1: 4 CPUs

{{< expand "simulation/scenario-5/node.yaml" >}}

{{< code-sample file="simulation/scenario-5/node.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f node.yaml
```

Create priority classes (low and high)

{{< expand "simulation/scenario-5/priority-classes.yaml" >}}

{{< code-sample file="simulation/scenario-5/priority-classes.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f priority-classes.yaml
```

Deploy a low-priority Pod

This allows any pod that matches this priority class to consume **3 CPUs** off the node.

{{< expand "simulation/scenario-5/low-priority-pod.yaml" >}}

{{< code-sample file="simulation/scenario-5/low-priority-pod.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f low-priority-pod.yaml
```

Deploy a high-priority pod

This allows any pod that matches this priority class to consume **3 CPUs** off the node.
Since the node has a maximum of **4 CPUs**, pods 
with a low-priority class that consumes over **1 CPU** will be preempted.

{{< expand "simulation/scenario-5/high-priority-pod.yaml" >}}

{{< code-sample file="simulation/scenario-5/high-priority-pod.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f high-priority-pod.yaml
```

Observe Preemption

Now, observe the preemption process. 
The higher priority pod will preempt the lower priority pod due to limited resources.

```bash
kubectl get pods

NAME                READY   STATUS    RESTARTS   AGE
high-priority-pod   1/1     Running   0          9m44s
```

You should see that the low-priority pod has been
terminated and the high-priority pod has been scheduled.

See more details about the preemption event

```bash
kubectl describe pod high-priority-pod | awk '/Events:/,/pod to node/'

Name:           high-priority-pod
Namespace:      limited-resources
Priority:       200
Node:           <node-name>
...
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  11m   default-scheduler  0/1 nodes are available: 1 Insufficient cpu.
  Normal   Scheduled         11m   default-scheduler  Successfully assigned default/high-priority-pod to node
```

The event log indicates that the high-priority pod preempted another pod due to its higher priority.

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling
scenario based on setting a [priority and preemption](https://kubernetes.io/docs/concepts/scheduling-eviction/) policy.

#### Scenario 6: Scheduling pods using pod topology spread constraints

Pod topology spread constraints scheduling policies can be applied in a KWOK cluster.

For this particular scenario, the cluster has 4 nodes that span 2 regions.
A pod topology spread constraint policy will be applied, so the pods are evenly scheduled across each region.

This image shows you what you should expect when testing this scenario.
You can follow the step-by-step guide after seeing this.

<img width="700px" src="simulation/scenario-6/README.svg">

Prerequisites

- KWOK must be installed on the machine. See [installation](https://kwok.sigs.k8s.io/docs/user/installation/).
- Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

Create cluster

```bash
kwokctl create cluster
```

View clusters

This ensures that the cluster was created successfully.

```bash
kwokctl get clusters
```

Deploy node

Below are the node resource specifications:

- Node: 4 worker nodes in the cluster
  - Node 1: 
    - Key = topology.kubernetes.io/region
    - Value = us-west-1
  - Node 2: 
    - Key = topology.kubernetes.io/region
    - Value = us-west-1
  - Node 3:
    - Key = topology.kubernetes.io/region
    - Value = us-west-2
  - Node 4:
    - Key = topology.kubernetes.io/region
    - Value = us-west-2

{{< expand "simulation/scenario-6/node-in-us-west-1.yaml" >}}

{{< code-sample file="simulation/scenario-6/node-in-us-west-1.yaml" >}}

{{< /expand >}}

{{< expand "simulation/scenario-6/node-in-us-west-2.yaml" >}}

{{< code-sample file="simulation/scenario-6/node-in-us-west-2.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f node-in-us-west-1.yaml
kubectl apply -f node-in-us-west-2.yaml
```

Create deployment

{{< expand "simulation/scenario-6/deployment.yaml" >}}

{{< code-sample file="simulation/scenario-6/deployment.yaml" >}}

{{< /expand >}}

```bash
kubectl apply -f deployment.yaml
```

The deployment creates 4 replicas, with its `maxSkew` set to 1.

Observe the topology spread

```bash
kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP         NODE     NOMINATED NODE   READINESS GATES
fake-app-685c7b9cc7-6rdgn   1/1     Running   0          13s   10.0.0.4   node-2   <none>           <none>
fake-app-685c7b9cc7-gcgsq   1/1     Running   0          13s   10.0.0.2   node-4   <none>           <none>
fake-app-685c7b9cc7-ssjq9   1/1     Running   0          13s   10.0.0.3   node-1   <none>           <none>
fake-app-685c7b9cc7-tqxrs   1/1     Running   0          13s   10.0.0.1   node-3   <none>           <none>
```

All pods are evenly distributed among each node across the regions.

Delete the cluster

```bash
kwokctl delete cluster
```

Conclusion

This example demonstrates how to use KWOK to simulate a scheduling
scenario based on setting a [pod topology spread constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/) policy.

Other scheduling scenarios can also be simulated using KWOK.
