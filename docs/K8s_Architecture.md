# K8s Architecture

A Kubernetes cluster consists of two main parts:

- Control Plane (Master Node) — the brain of the cluster that manages all operations and maintains the desired state.
- Worker Nodes — the machines where your applications (Pods) actually run.

---

## Control Plane
The Control Plane is responsible for managing the entire Kubernetes cluster. It decides where applications run, monitors their health, and maintains the desired state.

1. kube-api-server

The API Server is the entry point of Kubernetes. Every request passes through it.

Examples:
- kubectl commands
- REST API requests
- Dashboard requests

Example usage:

```bash
kubectl apply -f deployment.yaml
```

Flow:
```
kubectl → api server
```

Responsibilities:
- Receives all requests
- Authenticates users
- Authorizes requests
- Validates YAML files
- Stores cluster information in etcd
- Communicates with all Kubernetes components

Interview Point: API Server is the heart of Kubernetes because every component communicates through it.

2. etcd

The etcd database stores the complete state of the Kubernetes cluster. It stores:
- Pods
- Nodes
- Deployments
- Services
- Secrets
- ConfigMaps
- Namespaces

Think of it as Kubernetes' database.

Example flow:
```
You create a Deployment.
The Deployment specification is stored in etcd.

Deployment YAML
      ↓
API Server
     ↓
 etcd
```

Interview Point: etcd is the single source of truth for the Kubernetes cluster.

3. Scheduler

The Scheduler decides which Worker Node should run a newly created Pod. It considers:
- CPU availability
- Memory availability
- Node labels
- Taints & tolerations
- Affinity rules
- Resource requests

Example:
```
Node1 → CPU 20% Free
Node2 → CPU 70% Free
Node3 → CPU 10% Free
```
A new Pod requiring high CPU is most likely scheduled on Node2.

Flow:
```
Pod Created
      │
Scheduler
      │
Select Best Node
```

Interview Point: The Scheduler assigns Pods to the most suitable Worker Node.

4. Controller Manager

The Controller Manager continuously checks whether the cluster matches the desired state.

Example:
```
Desired state:
Replicas = 3
Current state:
Running Pods = 2
Controller detects the mismatch and creates another Pod.
```

Flow:
```
Desired = 3

Current = 2
Controller creates 1 more Pod
```

Controllers include:
- ReplicaSet Controller
- Deployment Controller
- Job Controller
- Node Controller
- Namespace Controller

Interview Point: Controller Manager continuously reconciles the desired state with the actual state.

5. Cloud Controller Manager

This component is mainly used when Kubernetes runs on cloud platforms like AWS, Azure, or GCP. It interacts with the Cloud Provider API.

Responsibilities:
- Create Load Balancers
- Attach storage volumes
- Manage cloud networking
- Handle node lifecycle

Flow:
```
Kubernetes
      │
Cloud Controller Manager
      │
AWS / Azure / GCP API
```

---

## Worker Node

Worker Nodes are responsible for running your application workloads. Each Worker Node contains:
- kubelet
- kube-proxy
- Container Runtime (CRI)
- Pods

1. kubelet

The kubelet is the Kubernetes agent running on every Worker Node. It communicates with the API Server.

Responsibilities:
- Receives Pod specifications
- Pulls container images
- Starts containers
- Monitors Pod health
- Reports status to the API Server

Flow:
```
API Server
      │
kubelet
      │
Container Runtime
      │
Pod Starts
```

Point: kubelet ensures that the containers described in Pod specifications are running and healthy.

2. kube-proxy

The kube-proxy manages networking on each Worker Node.

Responsibilities:
- Routes traffic to Pods
- Implements Services
- Performs load balancing across Pod replicas
- Maintains networking rules

Example (traffic distribution):
```
Three Pods:
Pod A
          Pod B
Pod C
Traffic:
User
   │
kube-proxy
 ├──► Pod A
 ├──► Pod B
 └──► Pod C
```

Point: kube-proxy enables communication between Services and Pods and distributes traffic.

3. CRI (Container Runtime Interface)

The CRI allows Kubernetes to communicate with the container runtime.

Common runtimes:
- containerd (most common today)
- CRI-O

Responsibilities:
- Pull images
- Create containers
- Start containers
- Stop containers

Point: Kubernetes does not run containers directly; it uses a container runtime through the CRI.

4. Pods

Pods are the smallest deployable units in Kubernetes. A Pod usually contains one container, but it can contain multiple tightly coupled containers that share networking and storage.

Example:
```
Pod
 ├── Nginx Container
 └── Log Sidecar Container
```

Pods are created and managed by higher-level objects like Deployments.

---

## Simple Diagram (textual)

Control Plane (HA)
[ Load Balancer ] -> [ kube-apiserver (N) ]
                     [ kube-controller-manager (N) ]
                     [ kube-scheduler (N) ]
                     [ etcd cluster (3/5/7) ]

Worker Nodes
[ Node A ] - kubelet, kube-proxy, container runtime, Pods
[ Node B ] - kubelet, kube-proxy, container runtime, Pods
[ Node C ] - kubelet, kube-proxy, container runtime, Pods

Services & Access
User -> kubectl / Ingress -> Service -> Pod(s)

---

This document was generated from user-provided notes and organized for clarity. You can add the diagram image to this file by uploading it to the repository and referencing it here.
