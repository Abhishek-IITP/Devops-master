# AWS Services for DevOps

AWS provides many services, but as a DevOps Engineer, we don't need to learn every AWS service initially.

The following are some of the **most commonly used and important AWS services for DevOps**.

---

# AWS Services We Should Know

## Compute

1. EC2
2. Lambda

## Networking

3. VPC

## Storage

4. EBS
5. S3

## Security & Identity

6. IAM
7. KMS

## Monitoring & Auditing

8. CloudWatch
9. CloudTrail
10. AWS Config

## Containers

11. ECR
12. ECS
13. Fargate
14. EKS

## CI/CD

15. CodePipeline
16. CodeBuild
17. CodeDeploy

## Cost Management

18. AWS Billing & Cost Management

---

# 1. Amazon EC2

## What is EC2?

EC2 stands for:

> **Elastic Compute Cloud**

Amazon EC2 provides scalable compute capacity in AWS.

In simple words:

> **EC2 is a Virtual Machine in AWS.**

When we launch an EC2 instance, AWS provides us with a virtual server on which we can install and run our applications.

---

## Why Do We Use EC2?

Suppose we have an application:

```text
My Application
      ↓
Needs a Server
      ↓
EC2 Instance
      ↓
Linux / Windows
      ↓
Application Running
```

Instead of purchasing a physical server, we can create an EC2 instance in a few minutes.

---

## What Can We Configure in EC2?

When creating an EC2 instance, we can choose:

* Operating System
* Instance Type
* CPU
* RAM
* Storage
* Network
* Security Groups
* IAM Role
* Key Pair

---

## EC2 Instance Type

The instance type determines the resources available to the EC2 instance.

Example:

```text
t3.micro
t3.small
t3.medium
m7i.large
c7i.large
r7i.large
```

Different instance families are designed for different workloads.

### General idea

```text
CPU Intensive
    ↓
Compute Optimized

Memory Intensive
    ↓
Memory Optimized

General Workload
    ↓
General Purpose
```

---

## Important EC2 Concepts

### AMI

AMI stands for:

> **Amazon Machine Image**

An AMI is a template used to launch an EC2 instance.

It can contain:

* Operating System
* Software
* Configuration
* Required packages

Example:

```text
Ubuntu AMI
    ↓
Launch EC2
    ↓
Ubuntu Server
```

---

### Security Group

A Security Group acts like a virtual firewall for an EC2 instance.

It controls:

* Incoming traffic
* Outgoing traffic

Example:

```text
Internet
   ↓
Port 80
   ↓
Security Group
   ↓
EC2
```

---

### Key Pair

A key pair is commonly used to securely connect to an EC2 instance.

For Linux:

```bash
ssh -i my-key.pem ubuntu@<public-ip>
```

---

## EC2 Scaling

EC2 can be scaled in two ways.

### Vertical Scaling

Increase the resources of an existing instance.

```text
2 CPU / 4 GB RAM
        ↓
4 CPU / 8 GB RAM
```

This is called:

> **Scale Up**

---

### Horizontal Scaling

Add more EC2 instances.

```text
        Load Balancer
             ↓
      ┌──────┼──────┐
      ↓      ↓      ↓
     EC2    EC2    EC2
```

This is called:

> **Scale Out**

---

## Common EC2 Interview Questions

### What is EC2?

EC2 is an AWS service that provides scalable virtual compute capacity.

### What is an AMI?

AMI is a template used to launch EC2 instances.

### What is a Security Group?

A Security Group is a virtual firewall that controls network traffic to and from an EC2 instance.

### What is the difference between vertical and horizontal scaling?

```text
Vertical Scaling
→ Increase resources of one machine

Horizontal Scaling
→ Add more machines
```

---

# 2. Amazon VPC

## What is VPC?

VPC stands for:

> **Virtual Private Cloud**

A VPC is a logically isolated virtual network in AWS where we can launch AWS resources.

In simple words:

> **VPC is our private network inside AWS.**

---

## Why Do We Need VPC?

Suppose we have:

```text
Internet
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Database
```

We don't want every resource to be directly accessible from the internet.

VPC allows us to design the network and control:

* IP addresses
* Subnets
* Routing
* Internet access
* Private communication
* Security

---

# Important VPC Components

## VPC

The main virtual network.

```text
VPC
├── Public Subnet
├── Private Subnet
├── Route Tables
├── Internet Gateway
├── NAT Gateway
└── Security Controls
```

---

## CIDR

CIDR defines the IP address range of the VPC.

Example:

```text
10.0.0.0/16
```

This defines the network address range available inside the VPC.

---

## Subnet

A subnet is a range of IP addresses inside a VPC.

Example:

```text
VPC: 10.0.0.0/16

├── Public Subnet
│   └── 10.0.1.0/24
│
└── Private Subnet
    └── 10.0.2.0/24
```

A subnet belongs to one Availability Zone.

---

## Public Subnet

A subnet that has a route to an Internet Gateway.

Typically used for:

* Load Balancers
* Bastion hosts
* Public-facing resources

---

## Private Subnet

A subnet that does not have a direct route from the internet through an Internet Gateway.

Typically used for:

* Application servers
* Databases
* Internal services

---

## Internet Gateway

An Internet Gateway allows resources in a VPC to communicate with the internet when routing and addressing are configured appropriately.

```text
Internet
    ↓
Internet Gateway
    ↓
Public Subnet
    ↓
EC2
```

---

## NAT Gateway

NAT Gateway allows resources in private subnets to make outbound connections to the internet without allowing unsolicited inbound internet connections to those resources.

Example:

```text
Private EC2
    ↓
NAT Gateway
    ↓
Internet
```

---

## Route Table

A Route Table determines where network traffic should go.

Example:

```text
0.0.0.0/0 → Internet Gateway
```

means traffic destined for the internet is sent through the Internet Gateway.

---

## Security Group vs NACL

### Security Group

* Works at the instance/ENI level
* Stateful
* Controls inbound and outbound traffic
* Allows rules

### Network ACL

* Works at the subnet level
* Stateless
* Supports inbound and outbound rules
* Rules can allow or deny traffic

### Easy Way to Remember

```text
Security Group
→ Instance level
→ Stateful

NACL
→ Subnet level
→ Stateless
```

---

## VPC Interview Questions

### What is VPC?

A VPC is a logically isolated virtual network in AWS.

### Public Subnet vs Private Subnet?

```text
Public Subnet
→ Route to Internet Gateway

Private Subnet
→ No direct inbound route from Internet Gateway
```

### What is an Internet Gateway?

It provides connectivity between a VPC and the internet when routing and addressing are configured for that purpose.

### What is NAT Gateway?

It allows resources in private subnets to initiate outbound internet connections.

### Security Group vs NACL?

```text
Security Group
→ Instance level
→ Stateful

NACL
→ Subnet level
→ Stateless
```

---

# 3. Amazon EBS

## What is EBS?

EBS stands for:

> **Elastic Block Store**

EBS provides persistent **block storage** for EC2 instances.

In simple words:

> **EBS is like a hard disk attached to an EC2 instance.**

---

## Example

```text
EC2
│
├── OS
├── Application
└── EBS Volume
       │
       └── Data
```

---

## Why Do We Need EBS?

Suppose our EC2 instance stores:

```text
application files
database files
logs
user data
```

We need persistent storage for this data.

EBS provides that storage.

---

## Important EBS Properties

* Persistent block storage
* Can be attached to EC2
* Can be backed up using snapshots
* Different volume types are available
* Volume size can be configured
* Volume performance can be configured depending on the volume type

---

## EBS vs Instance Store

### EBS

* Persistent
* Independent from the running life of the EC2 instance
* Can be backed up using snapshots

### Instance Store

* Temporary storage
* Physically associated with the host
* Data can be lost when the instance is stopped/terminated depending on the lifecycle

### Remember

```text
EBS
→ Persistent

Instance Store
→ Temporary
```

---

## EBS Interview Questions

### What is EBS?

EBS is persistent block-level storage that can be attached to EC2 instances.

### Can EBS survive EC2 termination?

It depends on the volume's DeleteOnTermination setting. The default behavior can vary based on how the volume was configured.

### What is an EBS Snapshot?

A snapshot is a point-in-time backup of an EBS volume.

---

# 4. Amazon S3

## What is S3?

S3 stands for:

> **Simple Storage Service**

S3 is an **object storage service**.

In simple words:

> **S3 is used to store files/objects in AWS.**

---

## Examples of Data Stored in S3

* Images
* Videos
* Documents
* Backups
* Logs
* Static websites
* Application artifacts
* Data lake objects

---

## S3 Structure

```text
S3
│
└── Bucket
     │
     ├── image.jpg
     ├── video.mp4
     ├── backup.zip
     └── logs/
```

---

## Bucket

A bucket is a container for objects.

Example:

```text
my-company-backups
```

Inside it:

```text
database-backup.zip
logs/
images/
```

---

## Object

An object is the actual data stored in S3.

An object consists conceptually of:

* Data
* Key
* Metadata

---

## Important S3 Features

* High durability
* Scalable object storage
* Versioning
* Lifecycle policies
* Encryption
* Access control
* Storage classes
* Event notifications

---

## S3 Versioning

Versioning allows multiple versions of an object to be maintained.

Example:

```text
app.zip
   ↓
Version 1
   ↓
Version 2
   ↓
Version 3
```

Useful for recovering from accidental overwrites or deletions.

---

## S3 Lifecycle

Lifecycle rules can automatically transition or expire objects.

Example:

```text
Day 0
↓
S3 Standard

After 30 days
↓
Cheaper Storage Class

After 1 year
↓
Archive

After required retention
↓
Delete
```

---

## S3 vs EBS

| S3                             | EBS                                      |
| ------------------------------ | ---------------------------------------- |
| Object storage                 | Block storage                            |
| Accessed through APIs          | Attached to compute instances            |
| Highly scalable object storage | Disk-like storage                        |
| Good for files/backups         | Good for OS/application/database storage |
| Not a normal filesystem disk   | Behaves like a block device              |

### Easy Way

```text
S3
→ Store Objects / Files

EBS
→ Disk for EC2
```

---

## S3 Interview Questions

### What is S3?

S3 is AWS object storage used to store and retrieve data.

### What is a Bucket?

A bucket is a container for S3 objects.

### What is an Object?

The actual data stored in S3.

### S3 vs EBS?

```text
S3 → Object Storage
EBS → Block Storage
```

---

# 5. IAM

## What is IAM?

IAM stands for:

> **Identity and Access Management**

IAM controls:

> **Who can access what in AWS?**

It handles:

* Authentication
* Authorization
* Permissions

---

## Simple Example

Suppose:

```text
Developer
    ↓
Needs access to S3
```

We can create an IAM policy allowing the developer to access specific S3 resources.

---

# IAM Components

## User

Represents an identity that can be used by a person or application in AWS.

---

## Group

A collection of IAM users.

Example:

```text
Developers Group
├── User A
├── User B
└── User C
```

---

## Role

A role is an identity with permissions that can be assumed by trusted entities.

Roles are heavily used in AWS automation.

Example:

```text
EC2
 ↓
IAM Role
 ↓
S3 Access
```

The application on EC2 can use the role instead of storing AWS access keys on the server.

---

## Policy

A policy defines permissions.

Example concept:

```text
Allow
S3
GetObject
bucket/example/*
```

---

## Authentication vs Authorization

### Authentication

> **Who are you?**

### Authorization

> **What are you allowed to do?**

Example:

```text
Login
 ↓
Authentication

Access S3
 ↓
Authorization
```

---

## IAM Best Practices

* Use least privilege
* Prefer IAM roles for AWS workloads
* Avoid using root user for daily work
* Enable MFA where appropriate
* Avoid hardcoding access keys
* Rotate credentials when credentials are used
* Give only required permissions

---

## IAM Interview Questions

### What is IAM?

IAM controls authentication and authorization for AWS resources.

### IAM User vs Role?

```text
User
→ Identity associated with a person/application

Role
→ Identity that can be assumed by trusted entities
```

### What is Least Privilege?

Give only the permissions required to perform a task.

### Why use IAM Roles with EC2?

To allow applications running on EC2 to access AWS services without storing long-term credentials on the instance.

---

# 6. AWS KMS

## What is KMS?

KMS stands for:

> **Key Management Service**

AWS KMS is used to create and control cryptographic keys used to protect data.

In simple words:

> **KMS helps us manage encryption keys.**

---

## Why Do We Need KMS?

Suppose we have:

```text
Sensitive Data
      ↓
Encryption
      ↓
Encrypted Data
```

We need an encryption key to encrypt and decrypt the data.

KMS helps us manage those keys.

---

## Common KMS Use Cases

KMS can be integrated with services such as:

* S3
* EBS
* RDS
* Secrets and other AWS services

Example:

```text
EBS
 ↓
KMS Key
 ↓
Encrypted Volume
```

---

## KMS Key

A KMS key is used in cryptographic operations and is managed by AWS KMS.

---

## Key Policy

A key policy controls who can use or manage a KMS key.

---

## KMS vs IAM

These are different.

```text
IAM
→ Who can access AWS resources?

KMS
→ How are encryption keys managed?
```

---

## KMS Interview Questions

### What is KMS?

KMS is AWS's managed key management service used to create and control cryptographic keys.

### Why is KMS used?

To protect data through encryption and centrally manage encryption keys.

### KMS vs IAM?

```text
IAM → Access control
KMS → Encryption key management
```

---

# 7. CloudWatch

## What is CloudWatch?

Amazon CloudWatch is an AWS monitoring and observability service.

It helps us monitor:

* Metrics
* Logs
* Alarms
* Dashboards
* Application performance
* Resource utilization

---

## Example

Suppose we have an EC2 server.

We want to know:

```text
CPU Usage
Memory
Disk
Network
Application Logs
```

CloudWatch can help us monitor these signals.

---

## CloudWatch Metrics

Metrics are numerical measurements.

Example:

```text
CPUUtilization = 85%
```

---

## CloudWatch Logs

Applications and AWS services can send logs to CloudWatch Logs.

Example:

```text
Application
     ↓
CloudWatch Logs
     ↓
Log Group
     ↓
Log Stream
```

---

## CloudWatch Alarm

An alarm can monitor a metric and trigger an action when a condition is met.

Example:

```text
CPU > 80%
     ↓
CloudWatch Alarm
     ↓
Action / Notification
```

---

## CloudWatch Dashboard

Dashboards provide a visual view of metrics and logs.

Example:

```text
EC2 CPU
EC2 Network
Lambda Errors
Application Logs
```

---

## CloudWatch vs CloudTrail

This is a VERY important interview question.

### CloudWatch

Used mainly for:

> **Monitoring and observability**

### CloudTrail

Used mainly for:

> **Auditing AWS API activity**

Easy way:

```text
CloudWatch
→ What is happening with my system?

CloudTrail
→ Who did what in my AWS account?
```

---

## CloudWatch Interview Questions

### What is CloudWatch?

A monitoring and observability service for AWS resources and applications.

### What is a CloudWatch Metric?

A numerical measurement of system or application behavior.

### What is a CloudWatch Alarm?

An alarm evaluates a metric against conditions and can trigger configured actions.

### CloudWatch vs CloudTrail?

```text
CloudWatch → Monitoring
CloudTrail  → Auditing/API activity
```

---

# 8. AWS CloudTrail

## What is CloudTrail?

CloudTrail records AWS API activity.

It can help answer:

> **Who performed an action, what action was performed, when it happened, and from where?**

---

## Example

Suppose someone deletes an EC2 instance.

CloudTrail can help identify:

```text
Who?
 ↓
IAM User / Role

What?
 ↓
TerminateInstances

When?
 ↓
Timestamp

Where from?
 ↓
Source IP / request context
```

---

## CloudTrail Records Actions From

* AWS Console
* AWS CLI
* AWS SDKs
* AWS APIs

---

## CloudTrail vs CloudWatch

```text
CloudWatch
→ System/Application Monitoring

CloudTrail
→ AWS API Auditing
```

### Example

If CPU is high:

```text
CloudWatch
```

If you want to know who deleted an EC2 instance:

```text
CloudTrail
```

---

## CloudTrail Interview Question

### What is CloudTrail?

CloudTrail is an AWS auditing service that records AWS API activity.

### CloudWatch vs CloudTrail?

> CloudWatch is primarily for monitoring and observability, while CloudTrail records and audits AWS API activity.

---

# 9. AWS Config

## What is AWS Config?

AWS Config provides information about the configuration of AWS resources and how their configurations change over time.

It can help answer:

> **What is the configuration of my AWS resources, and how did it change?**

---

## Example

Suppose an EC2 Security Group changes.

AWS Config can help track configuration changes and evaluate compliance using rules.

---

## CloudTrail vs Config

This is an important interview question.

### CloudTrail

Tracks:

> **API activity**

Example:

```text
Who changed Security Group?
```

### AWS Config

Tracks:

> **Resource configuration and configuration history**

Example:

```text
What was the Security Group configuration?
How did its configuration change?
```

### Easy Way

```text
CloudTrail
→ Who did what?

Config
→ What is/was the resource configuration?
```

---

# 10. Amazon ECR

## What is ECR?

ECR stands for:

> **Elastic Container Registry**

ECR is a managed container image registry.

In simple words:

> **ECR is like a private Docker image repository in AWS.**

---

## Why Do We Need ECR?

Suppose we build a Docker image:

```bash
docker build -t myapp .
```

We need somewhere to store it.

```text
Developer
    ↓
Docker Image
    ↓
ECR
    ↓
ECS / EKS / EC2
```

---

## Example

```text
ECR Repository
│
├── myapp:v1
├── myapp:v2
└── myapp:v3
```

ECS or EKS can pull the image from ECR to run the application.

---

## ECR Interview Question

### What is ECR?

ECR is AWS's managed container registry used to store, manage, and distribute container images.

### ECR vs S3?

```text
ECR
→ Container Images

S3
→ General Object Storage
```

---

# 11. Amazon ECS

## What is ECS?

ECS stands for:

> **Elastic Container Service**

ECS is AWS's managed container orchestration service.

In simple words:

> **ECS is used to run and manage containers on AWS.**

---

## ECS Architecture

```text
ECS Cluster
     │
     ├── ECS Service
     │      │
     │      ├── Task
     │      │    └── Container
     │      │
     │      └── Task
     │           └── Container
     │
     └── Other Services
```

---

# Important ECS Concepts

## Cluster

Logical grouping of ECS resources.

---

## Task Definition

Defines how a container should run.

It can specify:

* Container image
* CPU
* Memory
* Ports
* Environment variables
* IAM roles
* Logging

Example:

```text
Task Definition
      ↓
Image: myapp:v1
CPU: 512
Memory: 1GB
Port: 3000
```

---

## Task

A running instance of a Task Definition.

```text
Task Definition
      ↓
    Task
      ↓
Container
```

---

## Service

An ECS Service maintains the desired number of running tasks.

Example:

```text
Desired Count = 3

Task 1
Task 2
Task 3
```

If one task fails, the service can launch another task to maintain the desired count.

---

# ECS Launch Options

ECS workloads can run using different infrastructure models, including:

* EC2
* Fargate

---

# ECS on EC2

You manage the underlying EC2 instances.

```text
ECS
 ↓
EC2 Instances
 ↓
Containers
```

You are responsible for more of the underlying compute infrastructure.

---

# ECS on Fargate

AWS manages the underlying server infrastructure for the containers.

```text
ECS
 ↓
Fargate
 ↓
Containers
```

You specify the resources required for the task instead of managing the underlying servers.

---

# ECS Interview Questions

### What is ECS?

ECS is a managed container orchestration service used to deploy, manage, and scale containerized applications.

### What is a Task Definition?

A Task Definition is a configuration that describes how an ECS task/container should run.

### Task vs Task Definition?

```text
Task Definition
→ Blueprint

Task
→ Running instance of that blueprint
```

### What is an ECS Service?

An ECS Service maintains the desired number of tasks and manages their deployment and replacement.

---

# 12. AWS Fargate

## What is Fargate?

Fargate is a serverless compute engine for containers.

It can be used with:

* Amazon ECS
* Amazon EKS

In simple words:

> **Fargate lets you run containers without managing the underlying servers.**

---

## Traditional ECS on EC2

```text
You
 ↓
ECS
 ↓
EC2
 ↓
OS
 ↓
Container
```

You manage the EC2 infrastructure.

---

## ECS on Fargate

```text
You
 ↓
ECS
 ↓
Fargate
 ↓
Container
```

AWS manages the underlying compute infrastructure.

---

# Fargate vs EC2

| Fargate                          | EC2                                         |
| -------------------------------- | ------------------------------------------- |
| Serverless container compute     | Virtual machines                            |
| No server management             | You manage servers                          |
| Pay for requested task resources | Pay for EC2 resources                       |
| Easier operationally             | More control                                |
| Good for container workloads     | Useful when you need infrastructure control |

---

# Important Interview Question

## What is the difference between ECS and Fargate?

They are not exactly competitors.

```text
ECS
→ Container Orchestrator

Fargate
→ Container Compute Engine
```

ECS manages the container workload.

Fargate provides the compute environment for running containers without you managing servers.

---

# 13. Amazon EKS

## What is EKS?

EKS stands for:

> **Elastic Kubernetes Service**

EKS is AWS's managed Kubernetes service.

In simple words:

> **EKS allows us to run Kubernetes clusters on AWS while AWS manages the Kubernetes control plane and integrates it with AWS infrastructure.**

---

## EKS Architecture

Conceptually:

```text
                EKS Cluster
                    │
        ┌───────────┴───────────┐
        │                       │
 Control Plane              Worker Nodes
        │                       │
        │                 ┌─────┼─────┐
        │                 │     │     │
        │               Pod   Pod   Pod
```

---

# Why Use EKS?

Use EKS when you want Kubernetes features such as:

* Kubernetes deployments
* Services
* Ingress
* ConfigMaps
* Secrets
* StatefulSets
* Horizontal Pod Autoscaling
* Kubernetes ecosystem
* Kubernetes APIs

---

# EKS Compute Options

Kubernetes workloads in EKS can run on infrastructure such as:

* EC2
* Fargate
* Other supported EKS compute options

---

# EKS vs ECS

This is one of the **most important AWS interview questions**.

## ECS

AWS-native container orchestration service.

```text
Application
    ↓
Container
    ↓
ECS
```

## EKS

Managed Kubernetes service.

```text
Application
    ↓
Container
    ↓
Kubernetes
    ↓
EKS
```

---

## ECS vs EKS

| ECS                             | EKS                                        |
| ------------------------------- | ------------------------------------------ |
| AWS-native orchestration        | Managed Kubernetes                         |
| Simpler AWS-specific experience | Kubernetes ecosystem                       |
| Easier for teams focused on AWS | Useful for Kubernetes skills/ecosystem     |
| Less Kubernetes complexity      | More Kubernetes concepts                   |
| Uses ECS APIs/concepts          | Uses Kubernetes APIs                       |
| Integrates strongly with AWS    | Kubernetes is portable across environments |

### Easy Way to Remember

```text
ECS
→ AWS Container Orchestration

EKS
→ Kubernetes on AWS
```

---

# EKS vs ECS vs Fargate

Very important:

```text
ECS
→ Orchestrator

EKS
→ Kubernetes Orchestrator

Fargate
→ Serverless Container Compute
```

Example:

```text
Option 1:

ECS
 ↓
EC2
 ↓
Containers
```

```text
Option 2:

ECS
 ↓
Fargate
 ↓
Containers
```

```text
Option 3:

EKS
 ↓
EC2
 ↓
Pods
```

```text
Option 4:

EKS
 ↓
Fargate
 ↓
Pods
```

---

# 14. AWS CodePipeline

## What is CodePipeline?

CodePipeline is AWS's continuous delivery service.

It allows us to automate the software release process.

Example:

```text
Developer
   ↓
Git Repository
   ↓
CodePipeline
   ↓
Build
   ↓
Test
   ↓
Deploy
   ↓
Production
```

---

## Why Use CodePipeline?

It can automate:

* Source
* Build
* Test
* Deployment
* Approval stages

---

# Example Pipeline

```text
Source
  ↓
CodeBuild
  ↓
Test
  ↓
CodeDeploy
  ↓
Production
```

---

## CodePipeline Interview Question

### What is CodePipeline?

CodePipeline is a continuous delivery service that automates stages in the software release process.

### CodePipeline vs Jenkins?

```text
CodePipeline
→ AWS managed CI/CD orchestration

Jenkins
→ Open-source automation server
```

CodePipeline can integrate with AWS services and third-party tools.

---

# 15. AWS CodeBuild

## What is CodeBuild?

CodeBuild is a fully managed build service.

It can:

* Compile source code
* Run tests
* Build artifacts
* Build Docker images

---

## Example

```text
Source Code
    ↓
CodeBuild
    ↓
Install Dependencies
    ↓
Build
    ↓
Test
    ↓
Artifact / Docker Image
```

---

## CodeBuild Interview Question

### What is CodeBuild?

CodeBuild is a managed AWS service that compiles source code, runs tests, and produces deployable artifacts.

### CodeBuild vs CodePipeline?

```text
CodePipeline
→ Orchestrates the pipeline

CodeBuild
→ Performs build/test work
```

---

# 16. AWS CodeDeploy

## What is CodeDeploy?

CodeDeploy automates application deployments.

It can deploy applications to:

* EC2
* On-premises servers
* Lambda
* ECS

---

## Example

```text
New Application Version
        ↓
CodeDeploy
        ↓
EC2 / Lambda / ECS
        ↓
New Version Running
```

---

## Deployment Strategies

### In-Place Deployment

Update the existing environment.

```text
Old Version
    ↓
Update
    ↓
New Version
```

---

### Blue/Green Deployment

Create a new environment and shift traffic.

```text
          Load Balancer
              │
       ┌──────┴──────┐
       ↓             ↓
     Blue           Green
     OLD             NEW
```

After testing:

```text
Traffic
   ↓
Green
```

---

## CodeDeploy Interview Question

### What is CodeDeploy?

CodeDeploy is an AWS deployment service that automates application deployments to supported compute environments such as EC2, Lambda, and ECS.

### CodeDeploy vs CodeBuild?

```text
CodeBuild
→ Build/Test

CodeDeploy
→ Deploy
```

---

# 17. AWS Config

## What is AWS Config?

AWS Config provides a detailed view of AWS resource configurations and their relationships.

It helps us understand:

* Current configuration
* Configuration history
* Resource relationships
* Compliance

---

## Example

Suppose a Security Group changes:

```text
Before
Port 22 → Restricted

       ↓

Someone changes it

       ↓

Port 22 → 0.0.0.0/0
```

AWS Config can help track the configuration state and evaluate it against compliance rules.

---

# AWS Config vs CloudTrail

```text
CloudTrail
→ Records API activity

AWS Config
→ Tracks resource configuration
```

Example:

```text
Who changed the Security Group?
        ↓
CloudTrail

What was the Security Group configuration?
        ↓
AWS Config
```

---

# 18. AWS Billing & Cost Management

## Why is Billing Important for DevOps?

DevOps engineers don't just deploy infrastructure.

They should also understand:

> **How much does the infrastructure cost?**

AWS provides billing and cost management capabilities to:

* View costs
* Analyze spending
* Create budgets
* Monitor usage
* Optimize costs
* Understand invoices

---

## Example

Suppose our project has:

```text
EC2
EBS
S3
EKS
NAT Gateway
Load Balancer
```

Our monthly bill might increase because of:

* Running instances
* Storage
* Data transfer
* NAT Gateway usage
* Load balancers
* Container workloads

A DevOps engineer should identify unnecessary resources and optimize them.

---

# Cost Optimization Examples

### Stop unused EC2 instances

```text
Unused EC2
    ↓
Stop / Terminate
    ↓
Lower Cost
```

### Right-size instances

Instead of:

```text
16 CPU
64 GB RAM
```

when the application only needs:

```text
2 CPU
8 GB RAM
```

choose a smaller instance.

---

### Use appropriate storage

Don't keep every object in expensive storage classes forever.

Use lifecycle policies where appropriate.

---

### Monitor unused resources

Check for:

* Unused EBS volumes
* Unused Elastic IPs
* Idle EC2 instances
* Old snapshots
* Unused load balancers
* Unused NAT resources

---

# Important AWS DevOps Architecture

A typical application might look like:

```text
                         Internet
                            │
                            ▼
                      Load Balancer
                            │
                            ▼
                         VPC
                            │
                 ┌──────────┴──────────┐
                 │                     │
          Public Subnet          Private Subnet
                 │                     │
                 │               ┌─────┴─────┐
                 │               │           │
                 │             ECS/EKS     Database
                 │               │
                 │            Containers
                 │               │
                 └───────────────┘
                         │
                         ▼
                        S3
```

Security:

```text
IAM
KMS
Security Groups
NACLs
```

Monitoring:

```text
CloudWatch
CloudTrail
AWS Config
```

Container Registry:

```text
ECR
```

CI/CD:

```text
CodePipeline
     ↓
CodeBuild
     ↓
CodeDeploy
```

---

# Complete AWS CI/CD Example

Suppose we have a Node.js application.

Developer pushes code:

```text
Developer
    ↓
GitHub
    ↓
CodePipeline
    ↓
CodeBuild
    ↓
Run Tests
    ↓
Build Docker Image
    ↓
Push Image to ECR
    ↓
Deploy
    ↓
ECS / EKS
    ↓
Application Running
```

Monitoring:

```text
Application
    ↓
CloudWatch
    ↓
Metrics + Logs + Alarms
```

Auditing:

```text
AWS API Activity
    ↓
CloudTrail
```

Security:

```text
IAM
KMS
Security Groups
```

Infrastructure:

```text
VPC
├── Public Subnets
└── Private Subnets
```

---

# ELK vs AWS CloudWatch

ELK is important for DevOps, but:

> **ELK is not an AWS service.**

ELK stands for:

```text
E → Elasticsearch
L → Logstash
K → Kibana
```

It is a popular open-source logging/observability stack.

---

## ELK

```text
Application
     ↓
  Logstash
     ↓
Elasticsearch
     ↓
  Kibana
```

### Logstash

Collects and processes logs.

### Elasticsearch

Stores and searches logs.

### Kibana

Provides visualization and dashboards.

---

# CloudWatch vs ELK

| CloudWatch                    | ELK                                |
| ----------------------------- | ---------------------------------- |
| AWS managed service           | Stack of technologies              |
| Strong AWS integration        | Highly customizable                |
| Metrics + Logs + Alarms       | Mainly logs/search/visualization   |
| Less infrastructure to manage | More infrastructure to manage      |
| Native AWS monitoring         | Can collect logs from many sources |

### Easy Way

```text
CloudWatch
→ AWS-native monitoring

ELK
→ Log collection + search + visualization stack
```

AWS also offers managed OpenSearch services, which are commonly used for AWS-based search and log analytics.

---

# Most Important AWS Comparisons

## EC2 vs Lambda

```text
EC2
→ Manage a virtual server

Lambda
→ Run code without managing servers
```

### EC2

You manage more infrastructure.

### Lambda

AWS manages the underlying infrastructure and automatically handles scaling for Lambda executions.

---

# S3 vs EBS

```text
S3
→ Object Storage

EBS
→ Block Storage
```

Example:

```text
Images / Backups / Files
        ↓
       S3

EC2 Disk / Database Disk
        ↓
       EBS
```

---

# EBS vs Instance Store

```text
EBS
→ Persistent

Instance Store
→ Temporary
```

---

# IAM vs KMS

```text
IAM
→ Access / Permissions

KMS
→ Encryption Keys
```

---

# CloudWatch vs CloudTrail

```text
CloudWatch
→ Monitoring

CloudTrail
→ API Auditing
```

---

# CloudTrail vs AWS Config

```text
CloudTrail
→ Who did what?

AWS Config
→ What is/was the resource configuration?
```

---

# ECS vs EKS

```text
ECS
→ AWS-native container orchestration

EKS
→ Managed Kubernetes
```

---

# ECS vs Fargate

```text
ECS
→ Container Orchestrator

Fargate
→ Serverless Container Compute
```

They can be used together.

```text
ECS + Fargate
```

---

# EKS vs Fargate

```text
EKS
→ Kubernetes platform

Fargate
→ Serverless compute for supported container workloads
```

They can also be used together:

```text
EKS + Fargate
```

---

# ECR vs ECS

```text
ECR
→ Store Container Images

ECS
→ Run Container Workloads
```

Example:

```text
Docker Image
    ↓
ECR
    ↓
ECS
    ↓
Running Container
```

---

# CodePipeline vs CodeBuild vs CodeDeploy

This is VERY important.

```text
CodePipeline
→ Orchestrates the pipeline

CodeBuild
→ Build + Test

CodeDeploy
→ Deployment
```

Example:

```text
CodePipeline
      │
      ├── CodeBuild
      │      ↓
      │   Build/Test
      │
      └── CodeDeploy
             ↓
          Deployment
```

---

# VPC vs Security Group

```text
VPC
→ Network

Security Group
→ Firewall for resources/interfaces
```

---

# Public Subnet vs Private Subnet

```text
Public Subnet
→ Has route to Internet Gateway

Private Subnet
→ No direct inbound internet route
```

---

# NAT Gateway vs Internet Gateway

```text
Internet Gateway
→ VPC ↔ Internet connectivity

NAT Gateway
→ Private Subnet → Internet
```

---

# Quick AWS Revision

```text
EC2
→ Virtual Server / Compute

VPC
→ Virtual Network

EBS
→ Block Storage / EC2 Disk

S3
→ Object Storage

IAM
→ Identity + Permissions

KMS
→ Encryption Key Management

CloudWatch
→ Monitoring + Metrics + Logs + Alarms

CloudTrail
→ AWS API Auditing

AWS Config
→ Resource Configuration + Compliance

ECR
→ Container Image Registry

ECS
→ Container Orchestration

EKS
→ Managed Kubernetes

Fargate
→ Serverless Container Compute

CodePipeline
→ CI/CD Pipeline Orchestration

CodeBuild
→ Build + Test

CodeDeploy
→ Deployment

Billing & Cost Management
→ AWS Cost Monitoring + Optimization
```

---

# AWS Services by Category

```text
COMPUTE
│
├── EC2
└── Lambda

NETWORKING
│
└── VPC

STORAGE
│
├── EBS
└── S3

SECURITY
│
├── IAM
└── KMS

MONITORING / AUDITING
│
├── CloudWatch
├── CloudTrail
└── AWS Config

CONTAINERS
│
├── ECR
├── ECS
├── EKS
└── Fargate

CI/CD
│
├── CodePipeline
├── CodeBuild
└── CodeDeploy

COST
│
└── AWS Billing & Cost Management
```

---

# One-Line Interview Revision

```text
EC2      → Compute
VPC      → Network
EBS      → Block Storage
S3       → Object Storage
IAM      → Access Control
KMS      → Encryption Keys
CloudWatch → Monitoring
CloudTrail  → Auditing
Config      → Resource Configuration
ECR      → Container Images
ECS      → Container Orchestration
EKS      → Kubernetes
Fargate  → Serverless Containers
Pipeline → CI/CD Orchestration
Build    → Build/Test
Deploy   → Deployment
Billing  → Cost Management
```

# Most Important Interview Questions to Prepare

1. What is EC2?
2. What is an AMI?
3. What is a Security Group?
4. What is VPC?
5. Public Subnet vs Private Subnet?
6. Internet Gateway vs NAT Gateway?
7. Security Group vs NACL?
8. What is EBS?
9. EBS vs Instance Store?
10. EBS vs S3?
11. What is S3?
12. What is S3 Versioning?
13. What is IAM?
14. IAM User vs IAM Role?
15. Authentication vs Authorization?
16. What is Least Privilege?
17. What is KMS?
18. IAM vs KMS?
19. What is CloudWatch?
20. What is CloudTrail?
21. CloudWatch vs CloudTrail?
22. CloudTrail vs AWS Config?
23. What is ECR?
24. ECR vs S3?
25. What is ECS?
26. What is an ECS Task Definition?
27. Task vs Task Definition?
28. What is an ECS Service?
29. What is Fargate?
30. ECS vs Fargate?
31. What is EKS?
32. ECS vs EKS?
33. EKS vs Fargate?
34. What is CodePipeline?
35. CodePipeline vs Jenkins?
36. What is CodeBuild?
37. CodeBuild vs CodePipeline?
38. What is CodeDeploy?
39. CodeBuild vs CodeDeploy?
40. What is Blue/Green Deployment?
41. What is AWS Config?
42. Why is cost optimization important in DevOps?
43. What is the difference between ELK and CloudWatch?
44. How would you design a CI/CD pipeline using AWS services?
45. How would you deploy a Docker application on AWS?
46. How would you monitor an application running on EC2?
47. How would you securely allow an EC2 instance to access S3?
48. How would you deploy containers without managing servers?
49. How would you deploy Kubernetes workloads on AWS?
50. How would you reduce AWS infrastructure costs?

---

# Final Mental Model

Remember AWS from a DevOps perspective like this:

```text
                    AWS
                     │
        ┌────────────┼────────────┐
        │            │            │
     COMPUTE      NETWORK       STORAGE
        │            │            │
     EC2/Lambda     VPC         S3/EBS
        │
        ▼
    CONTAINERS
        │
   ┌────┼────┐
   │    │    │
  ECR   ECS  EKS
          │    │
          └─┬──┘
            │
         Fargate

        SECURITY
            │
       IAM + KMS

      OBSERVABILITY
            │
   CloudWatch + CloudTrail
            │
         Config

          CI/CD
            │
  CodePipeline
       │
  ┌────┼─────┐
  ↓    ↓     ↓
Build Test Deploy
 │          │
CodeBuild  CodeDeploy
```

> **The easiest way to remember AWS for DevOps is not to memorize isolated services. Understand how they work together to build, secure, deploy, monitor, and operate an application.**
