# Kubernetes Interview Questions

## 1. What is the difference between Docker and Kubernetes?

**Answer:**

Docker is a containerization platform used to **build, package, and run containers**.

Kubernetes (K8s) is a **container orchestration platform** used to manage containers across multiple machines.

Kubernetes provides features such as:

- Container deployment
- Scaling
- Self-healing
- Load balancing
- Service discovery
- Rolling updates
- Cluster management

**Simple way to remember:**

> Docker helps us **run containers**, while Kubernetes helps us **manage containers at scale**.

---

## 2. What are the main components of Kubernetes architecture?

**Answer:**

Kubernetes mainly consists of two parts:

1. **Control Plane** - manages the overall Kubernetes cluster.
2. **Worker Nodes** - run the actual applications.

### Control Plane

The main components are:

- **API Server** - handles communication with the Kubernetes cluster.
- **etcd** - stores the cluster's configuration and state.
- **Controller Manager** - makes sure the cluster stays in the desired state.
- **Scheduler** - decides which worker node should run a Pod.
- **Cloud Controller Manager** - manages cloud-specific resources when Kubernetes is running on a cloud provider.

### Worker Node

The main components are:

- **Kubelet** - manages Pods on the node and communicates with the control plane.
- **Container Runtime** - runs containers.
- **Kube-proxy** - manages networking rules for Kubernetes Services.

**Simple way to remember:**

> Control Plane = **Manages the cluster**  
> Worker Node = **Runs the applications**

---

## 3. What is the main difference between Docker Swarm and Kubernetes?

**Answer:**

Docker Swarm and Kubernetes are both **container orchestration platforms**.

Docker Swarm is simpler and lightweight, so it can be easier to set up and learn.

Kubernetes provides more features, flexibility, and customization for managing larger and more complex applications.

### Main difference

| Docker Swarm | Kubernetes |
|---|---|
| Simple to set up | More complex to set up |
| Lightweight | More feature-rich |
| Easier for small setups | Suitable for complex deployments |
| Fewer built-in features | Large ecosystem and many features |

**Interview answer:**

> Docker Swarm is a simple and lightweight container orchestration tool, while Kubernetes is a more feature-rich and highly customizable platform commonly used for managing complex containerized applications.

---

## 4. What is the difference between a Docker container and a Kubernetes Pod?

**Answer:**

A **Docker container** is a running instance of a container image.

A **Pod** is the smallest deployable unit in Kubernetes. A Pod can contain **one or more containers** that share networking and storage.

Most commonly, a Pod contains one main application container.

**Simple way to remember:**

> Container = runs the application  
> Pod = Kubernetes unit that manages one or more containers

---

## 5. What is a Namespace in Kubernetes?

**Answer:**

A Namespace provides **logical isolation** inside a Kubernetes cluster.

It allows us to separate resources such as Pods, Services, and Deployments within the same cluster.

For example, we can have:

- `development`
- `testing`
- `production`

All of them can exist in the same Kubernetes cluster.

**Simple way to remember:**

> Namespace is like a **separate logical room** inside the same Kubernetes cluster.

---

## 6. What is the role of kube-proxy?

**Answer:**

`kube-proxy` is responsible for handling **network traffic for Kubernetes Services**.

It maintains network rules on worker nodes so that requests sent to a Service can reach the correct Pods.

For example:

```text
User
  |
  v
Kubernetes Service
  |
  v
kube-proxy / network rules
  |
  v
Pod
```

**Simple way to remember:**

> `kube-proxy` helps route Service traffic to the correct Pods.

---

## 7. What are the different types of Services in Kubernetes?

**Answer:**

The main Kubernetes Service types are:

1. **ClusterIP**
   - Default Service type.
   - Makes the application accessible inside the Kubernetes cluster.
   - It is commonly used for communication between applications.

2. **NodePort**
   - Exposes the Service through a port on each worker node.
   - Can be accessed using: `<Node-IP>:<NodePort>`

3. **LoadBalancer**
   - Exposes the Service externally using a cloud provider's load balancer.
   - Commonly used when running Kubernetes on cloud platforms.

4. **ExternalName**
   - Maps a Kubernetes Service to an external DNS name.
   - It does not create a normal Kubernetes network endpoint.

**Important:**

> Features like auto-scaling and self-healing are Kubernetes capabilities, not Service types.

---

## 8. What is the difference between NodePort and LoadBalancer Service?

**Answer:**

Both can expose an application outside the Kubernetes cluster, but they work differently.

### NodePort

NodePort exposes the Service through a specific port on each worker node.

Example:

```text
http://<Node-IP>:30080
```

Traffic can then be forwarded to the application's Pods.

### LoadBalancer

LoadBalancer creates or uses an external load balancer, usually provided by a cloud platform.

Example:

```text
Internet
   |
   v
Load Balancer
   |
   v
Kubernetes Service
   |
   v
Pods
```

**Simple difference:**

- **NodePort**: Access the application using a port on the node.
- **LoadBalancer**: Use an external load balancer to expose the application.

---

## 9. What is the role of Kubelet?

**Answer:**

Kubelet is an agent that runs on every worker node.

Its main responsibilities are:

- Manages Pods assigned to the node.
- Communicates with the Kubernetes API Server.
- Works with the container runtime to run containers.
- Monitors the health of Pods and containers.
- Reports the node and Pod status to the control plane.

**Simple way to remember:**

> Kubelet makes sure that the Pods assigned to its node are running as expected.

---

## 10. What are the day-to-day activities of a Kubernetes engineer?

**Answer:**

Some common day-to-day activities include:

### 1. Monitoring and Observability
- Checking application and cluster health.
- Monitoring CPU, memory, and other resources.
- Checking logs and alerts.

### 2. Troubleshooting
- Investigating failed Pods.
- Checking logs.
- Debugging networking or deployment issues.
- Fixing configuration problems.

### 3. Deployment and Lifecycle Management
- Deploying applications.
- Updating application versions.
- Managing Deployments and Services.
- Performing rollouts and rollbacks.

### 4. Cluster Maintenance
- Managing worker nodes.
- Checking cluster resources.
- Updating configurations.
- Managing Kubernetes resources.

**Simple interview answer:**

> A Kubernetes engineer usually works on deployments, monitoring, troubleshooting, cluster maintenance, scaling applications, and managing Kubernetes resources such as Pods, Services, and Deployments.
