# Introduction to Kubernetes Ingress

## What is the problem with Kubernetes Services?

A Kubernetes Service gives a stable way to reach a group of Pods.

For example, we can expose a Service outside the cluster by using:

- `NodePort`: opens a port on every worker node.
- `LoadBalancer`: asks the cloud provider to create a public load balancer.

These options work, but they can create problems when an application has many services:

1. Each application may need a separate public IP address.
2. Each public load balancer may increase the cloud bill.
3. Services do not provide advanced web traffic rules by themselves.
4. It is harder to manage HTTPS, domains, paths, and access rules for every service separately.

## How did companies handle this before Ingress?

Companies commonly used an enterprise load balancer, such as NGINX, F5, or a cloud load balancer.

The load balancer received traffic from users and decided where to send it. It could provide features such as:

- **Path-based routing:** send `/payments` to the payments application and `/orders` to the orders application.
- **Domain-based routing:** send `shop.example.com` to one application and `api.example.com` to another.
- **Ratio-based routing:** send more traffic to a stronger server.
- **Sticky sessions:** send the same user to the same server for a period of time.
- **HTTPS support:** manage certificates and encrypted traffic.
- **Allow and deny rules:** allow or block selected ports, IP addresses, or paths.

Without Ingress, these rules often have to be configured outside Kubernetes or repeated for many Services.

## The big cost problem

Imagine an application with 1,000 Services. If every Service uses `type: LoadBalancer`, the cloud provider may create 1,000 public load balancers or public IP addresses.

This causes two problems:

- The setup becomes difficult to manage.
- The cloud provider may charge for every load balancer or public IP address.

Ingress helps by using one common entry point for many applications.

## What is Kubernetes Ingress?

Ingress is a Kubernetes object that describes how outside HTTP and HTTPS traffic should reach Services inside the cluster.

It is like a set of traffic rules. For example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
	name: application-ingress
spec:
	rules:
		- host: example.com
			http:
				paths:
					- path: /orders
						pathType: Prefix
						backend:
							service:
								name: orders-service
								port:
									number: 80
```

This rule means:

> When a user visits `example.com/orders`, send the request to `orders-service` on port 80.

## Problems Ingress solves

Ingress can help us:

- Use one public entry point for many Services.
- Route traffic using a domain name.
- Route traffic using a URL path.
- Manage HTTPS certificates at one common place.
- Reduce the number of public load balancers and IP addresses.
- Keep traffic rules in Kubernetes configuration files.
- Apply common rules such as redirects, access limits, and traffic control.

## Ingress architecture

The request flow usually looks like this:

```text
User
	|
	v
Public IP or Cloud Load Balancer
	|
	v
Ingress Controller
	|
	v
Kubernetes Service
	|
	v
Pods
```

Step by step:

1. A user sends a request to a domain such as `example.com/orders`.
2. DNS points the domain to the public IP of the load balancer or Ingress Controller.
3. The Ingress Controller receives the request.
4. It reads the Ingress rules and checks the domain and path.
5. It forwards the request to the correct Kubernetes Service.
6. The Service sends the request to one of its healthy Pods.
7. The response travels back to the user through the same path.

## What is an Ingress Controller?

An Ingress resource only contains traffic rules. It does not route traffic by itself.

An **Ingress Controller** is the software that reads those rules and actually handles the traffic. It watches the Kubernetes API for Ingress resources and updates its configuration when the rules change.

Common Ingress Controllers include:

- NGINX Ingress Controller
- Traefik
- HAProxy
- Kong
- Cloud-provider controllers, such as AWS, Azure, or Google Cloud controllers

The controller usually runs inside the Kubernetes cluster as one or more Pods. It may be exposed through a `LoadBalancer` or `NodePort` Service so that users can reach it from outside the cluster.

## Ingress resource vs Ingress Controller

These two terms are different:

- **Ingress resource:** the configuration that says where traffic should go.
- **Ingress Controller:** the software that follows this configuration and forwards traffic.

Both are needed. Creating an Ingress resource without installing a compatible Ingress Controller will not route traffic.

## Important note

Ingress mainly handles HTTP and HTTPS traffic. For other protocols, such as some database or custom TCP traffic, use a Service of type `LoadBalancer`, `NodePort`, or a controller feature that supports that protocol.

