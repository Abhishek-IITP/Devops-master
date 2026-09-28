# Kubernetes Services - Detailed Notes

## 1) What if there is no Service in Kubernetes?

In Kubernetes, Pods are not stable endpoints.

- Pods are ephemeral.
- A Pod can be restarted, rescheduled, or replaced by a ReplicaSet/Deployment.
- When a new Pod is created, it gets a new IP address.
- Therefore, if an application or another service tries to talk to a Pod using its raw IP, that connection will break when the Pod changes.

Example:

- Deployment creates 3 Pods for an app.
- Pod A has IP 10.244.0.12
- Pod B has IP 10.244.0.15
- After restart or replacement, the new Pod may get IP 10.244.0.27

If the client is still using the old IP, it will fail.

So, without a Service, there is no stable network identity for a set of Pods.

---

## 2) Why do we need Services?

A Service gives a stable IP and DNS name to a group of Pods.

It acts as a fixed entry point for applications, even when the backend Pods keep changing.

### Main reasons:

1. Stable networking
   - A Service has a permanent ClusterIP.
   - Clients connect to the Service, not to individual Pods.

2. Load balancing
   - If multiple Pods are behind one Service, the Service distributes traffic across them.

3. Service discovery
   - Kubernetes uses labels and selectors to identify which Pods belong to a Service.

4. External exposure
   - Services can expose application internally or externally depending on type.

5. Abstraction
   - Users do not need to know exact Pod IPs.

---

## 3) Why Pod IPs change and how Service solves this

When a Pod dies, Kubernetes may create a new Pod due to ReplicaSet or Deployment.

- Old Pod is terminated.
- New Pod is created.
- New Pod gets a different IP.

If an app is hardcoded to use the old Pod IP, it will fail.

Service solves this by creating a stable virtual IP that always points to the group of matching Pods.

### Simple idea:

- Deployment manages Pods
- Service selects those Pods using labels
- kube-proxy updates routing rules
- Client talks to the Service IP, not to the underlying Pod IP

---

## 4) Service + load balancing

The Service is not just a static IP. It also performs traffic distribution.

### What happens behind the scenes?

- Service selects Pods using labels and selectors.
- kube-proxy watches the Service and Endpoints objects.
- kube-proxy updates network rules (iptables or IPVS) so requests to the Service IP are forwarded to the correct Pod(s).

This is how Kubernetes provides basic load balancing.

### Example:

If a Deployment has 3 replicas of a web app:

- Pod-1: 10.244.0.12
- Pod-2: 10.244.0.18
- Pod-3: 10.244.0.22

A Service called web-service has IP 10.96.0.10.

When a request comes to 10.96.0.10, kube-proxy forwards it to one of the three Pods.

So the user only sees one stable service endpoint instead of many dynamic Pod IPs.

---

## 5) Service discovery using labels and selectors

This is a very important concept in Kubernetes.

### Labels
Labels are key-value pairs attached to Kubernetes objects such as Pods, Deployments, Services, etc.

Example:

```yaml
metadata:
  labels:
    app: web
    tier: frontend
```

### Selectors
A Service uses a selector to choose which Pods it should route traffic to.

Example:

```yaml
spec:
  selector:
    app: web
```

This means:

- Any Pod with label `app=web` is included in this Service.
- Service creates an Endpoints object listing those Pods.
- kube-proxy uses that Endpoints list to route traffic.

### Why this matters

- Pods may be created or replaced dynamically.
- The Service still discovers the current matching Pods automatically.
- This is called service discovery.

---

## 6) What is kube-proxy?

kube-proxy is a Kubernetes agent running on each node.

It watches Services and Endpoints and configures forwarding rules.

### Purpose:

- maintain network rules for Service IPs
- connect client requests to backend Pods
- implement load balancing logic

### Two common modes:

1. iptables mode
   - uses Linux iptables rules
   - simple and common

2. IPVS mode
   - more advanced and efficient load balancing
   - better for high traffic and large-scale clusters

So the Service is a virtual abstraction, and kube-proxy is the component that actually routes traffic to the correct Pod.

---

## 7) Types of Service

Kubernetes provides different Service types depending on exposure needs.

### 1. ClusterIP (default)

This is the most common type.

- Internal only
- Accessible only within the cluster
- You get a stable virtual IP inside the cluster network

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

Use case:

- one microservice talking to another microservice inside the cluster

---

### 2. NodePort

This exposes the Service on each node's IP at a static port.

- accessible from within the organization or network if node IP is reachable
- each node listens on the same port
- Kubernetes forwards traffic to the Service and then to backend Pods

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

Access pattern:

```bash
http://<Node-IP>:30080
```

Use case:

- sometimes used for quick testing or internal access
- not the preferred production method compared to Ingress or Cloud LoadBalancer

---

### 3. LoadBalancer

This is used to expose the Service outside the cluster to the outside world.

- cloud provider provisions a cloud load balancer
- service gets a public IP or external IP
- traffic is forwarded to the cluster

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
```

Use case:

- public or internet-facing application
- production environments on cloud platforms

### What is the external IP?

On cloud providers like AWS, Azure or GCP, Kubernetes uses the cloud provider integration to create a cloud resource such as:

- AWS ELB / ALB / NLB
- Azure Load Balancer
- GCP Load Balancer

The Kubernetes Cloud Controller Manager (CCM) helps create and manage this external resource.

---

## 8) Role of Cloud Controller Manager (CCM) in generating ELB public IP

Cloud Controller Manager is a Kubernetes component that integrates Kubernetes with the cloud provider.

Its job is to manage cloud-specific resources like:

- Load balancers
- Node objects
- Routes
- Network addresses

### In the context of LoadBalancer Service:

When you create a Service of type LoadBalancer, Kubernetes asks the cloud provider to provision an external load balancer.

Then the Cloud Controller Manager:

1. creates the cloud load balancer resource
2. configures security groups/firewall rules if needed
3. attaches the cluster nodes or backend targets
4. assigns a public IP or DNS name
5. updates the Kubernetes Service with the external IP

### Example with AWS:

- Kubernetes sees a `LoadBalancer` Service
- CCM talks to AWS API
- AWS creates an ELB or NLB
- AWS gives a public DNS/IP
- Kubernetes updates the Service status with the external endpoint

So the CCM is the bridge between Kubernetes and the cloud provider.

Without CCM, Kubernetes may not know how to create external cloud infrastructure.

---

## 9) Why LoadBalancer does not work on Minikube

Minikube is a local single-node or small local Kubernetes environment.

It does not have full support for cloud-native load balancers because it is not running on a public cloud provider.

### Reasons:

1. No cloud provider integration
   - Minikube is not running on AWS, Azure, or GCP.
   - There is no real ELB or cloud load balancer provisioned.

2. No real public IP
   - In cloud environments, CCM creates a public network resource.
   - In Minikube, that cloud layer is not present.

3. Minikube uses a local driver
   - It runs on a VM or local container runtime.
   - The load balancer service is simulated instead of using a real cloud load balancer.

### What Minikube does instead

It usually supports a simulated or local load balancer behavior via a tunnel or a special service type handling, but it is not the same as a real cloud ELB.

For example, in Minikube:

- `minikube tunnel` may be used to expose services on localhost
- NodePort is more common for local testing
- `LoadBalancer` type may appear as pending because no cloud controller is managing it

### In simple words:

A LoadBalancer service requires a cloud-managed environment. Minikube is local, not cloud-managed, so a real external load balancer cannot be created there.

---

## 10) Diagram idea

```text
Client --> Service (ClusterIP / NodePort / LoadBalancer)
              |
              v
        kube-proxy
              |
              v
       Endpoints --> Pod-1
                   Pod-2
                   Pod-3
```

The client never talks directly to Pods.
It talks to the Service, and the Service forwards traffic to one or more matching Pods.

---

## 11) Summary

### Service is needed because:

- Pods have IPs that change
- ReplicaSet creates new Pods dynamically
- We need stable access to pods

### Service provides:

1. Load balancing
2. Service discovery via labels and selectors
3. Internal or external exposure

### Service types:

- ClusterIP: internal access only
- NodePort: access through node IP and port
- LoadBalancer: external access via cloud load balancer

### Important components:

- Deployment manages Pod replicas
- Service selects Pods
- Endpoints stores selected Pod IPs
- kube-proxy routes traffic
- Cloud Controller Manager creates external cloud load balancer resources

### Final answer to the original question:

Without a Service, Pods can come and go and their IPs change; this breaks connectivity. A Service provides a stable endpoint and load balancing across Pods. It is the standard way Kubernetes exposes applications, discovers them, and routes traffic reliably.

---

## 12) Short exam-style points

- Pods are ephemeral, so they should not be accessed directly.
- Service provides a stable IP and DNS name.
- Labels are used to identify Pods.
- Selectors are used by Service to match Pods.
- kube-proxy routes traffic to matching Pods.
- ClusterIP is used for internal communication.
- NodePort exposes a service on node IP/port.
- LoadBalancer provisions external cloud load balancer.
- CCM creates cloud load balancer resources such as ELB.
- Minikube does not support real cloud LoadBalancer because it lacks cloud provider integration.

---

## 13) Example YAML for all three main service types

### ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-clusterip
spec:
  selector:
    app: app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

### NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-nodeport
spec:
  type: NodePort
  selector:
    app: app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
```

### LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-lb
spec:
  type: LoadBalancer
  selector:
    app: app
  ports:
    - port: 80
      targetPort: 8080
```

---

## 14) One-line understanding

A Service in Kubernetes is the stable network entry point for a group of Pods, and it provides load balancing, service discovery, and external access while hiding the dynamic nature of Pod IPs.
