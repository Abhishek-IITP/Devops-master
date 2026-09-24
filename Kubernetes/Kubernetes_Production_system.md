what is kubernetes distrubution

EKS
openshift
tanzo
# Kubernetes Production Systems

## What Is a Kubernetes Distribution?

A Kubernetes distribution is Kubernetes packaged with additional components, tools, security defaults, support, and an installation or upgrade process. The upstream Kubernetes project provides the core platform; a distribution makes it easier to operate in a particular environment.

There is no universal priority order. Choose based on ownership, cloud provider, support requirements, budget, and how much infrastructure you want to manage:

| Option | Type | Typical use |
| --- | --- | --- |
| **Upstream Kubernetes** | Self-managed | Maximum control; you manage the control plane and worker nodes. |
| **Amazon EKS** | Managed AWS Kubernetes | AWS manages much of the control plane; you still manage worker capacity and workloads. |
| **Azure AKS** | Managed Azure Kubernetes | Azure-integrated managed Kubernetes. |
| **Google GKE** | Managed Google Cloud Kubernetes | Google-integrated managed Kubernetes. |
| **Red Hat OpenShift** | Enterprise Kubernetes platform | Kubernetes with an integrated developer platform, security, and Red Hat support. |
| **Rancher / SUSE Rancher** | Management platform | Centralized management of multiple Kubernetes clusters. |
| **Tanzu** | VMware enterprise platform | Kubernetes for VMware and enterprise environments. |
| **DigitalOcean Kubernetes** | Managed Kubernetes | A simpler managed option on DigitalOcean. |

## What Is KOPS?

**KOPS (Kubernetes Operations)** is a command-line tool for creating and managing self-managed Kubernetes clusters on AWS. It generates the cluster configuration, stores that configuration in an S3 state store, and creates AWS infrastructure such as the control plane, worker nodes, networking, security groups, and DNS records.

KOPS is useful for learning and for teams that need control over the AWS infrastructure. It is not automatically the best production choice: managed services such as EKS reduce control-plane maintenance and usually reduce operational work.

## KOPS Installation Prerequisites

Use an Ubuntu machine, an EC2 instance, or a local computer with AWS access. Install:

- Python 3
- AWS CLI
- `kubectl`
- `kops`

The commands below install `kubectl` from the Kubernetes package repository on Ubuntu:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl apt-transport-https gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key \
	| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' \
	| sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubectl
```

Install the AWS CLI and KOPS:

```bash
sudo snap install aws-cli --classic

KOPS_VERSION=$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest \
	| grep tag_name | cut -d '"' -f 4)
curl -LO "https://github.com/kubernetes/kops/releases/download/${KOPS_VERSION}/kops-linux-amd64"
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops

aws --version
kubectl version --client
kops version
```

## AWS Permissions and Configuration

Configure credentials for the AWS account and region you intend to use:

```bash
aws configure
aws sts get-caller-identity
```

For a learning environment, the identity needs permissions for EC2, S3, IAM, VPC, and Route 53. In production, use a dedicated IAM role with least-privilege permissions instead of broad administrator-style policies such as `AmazonEC2FullAccess`.

## Step-by-Step Cluster Setup

The example uses:

```bash
export AWS_REGION=us-east-1
export KOPS_STATE_STORE=s3://kops-abhi-storage
export CLUSTER_NAME=k8s.example.com
export DNS_ZONE=example.com
```

Replace `example.com` with a domain you own. The domain must be registered, and its nameservers must point to the Route 53 hosted zone.

### 1. Create the KOPS state bucket

KOPS stores cluster configuration and state in S3.

> **AWS COST WARNING:** S3 storage and requests may incur charges. Bucket names are globally unique.

```bash
aws s3api create-bucket \
	--bucket kops-abhi-storage \
	--region "$AWS_REGION"
```

For regions other than `us-east-1`, add:

```bash
--create-bucket-configuration LocationConstraint="$AWS_REGION"
```

### 2. Configure a custom Route 53 domain

If the hosted zone does not already exist, create one:

> **AWS COST WARNING:** Route 53 charges for hosted zones and DNS queries. Domain registration is a separate charge.

```bash
aws route53 create-hosted-zone \
	--name "$DNS_ZONE" \
	--caller-reference "kops-$(date +%s)"
```

Get the hosted zone nameservers:

```bash
aws route53 get-hosted-zone --id <HOSTED_ZONE_ID>
```

Update the domain registrar with the four Route 53 nameservers returned by that command. If the domain already uses a Route 53 hosted zone, skip hosted-zone creation and use its zone name.

### 3. Create the cluster configuration

This command writes the desired cluster configuration to the KOPS state store. It does not create all EC2 resources yet.

```bash
kops create cluster \
	--name="$CLUSTER_NAME" \
	--state="$KOPS_STATE_STORE" \
	--dns-zone="$DNS_ZONE" \
	--zones="$AWS_REGION"a \
	--node-count=1 \
	--node-size=t3.small \
	--master-size=t3.small \
	--master-volume-size=20 \
	--node-volume-size=20
```

Review the generated configuration before applying it:

```bash
kops edit cluster "$CLUSTER_NAME"
kops get cluster --name "$CLUSTER_NAME" --state "$KOPS_STATE_STORE"
```

The example uses small instances for learning only. They may be insufficient for production workloads and are not guaranteed to be free tier eligible.

### 4. Build the cluster

> **AWS COST WARNING:** This command creates billable AWS resources, including EC2 instances, EBS volumes, networking resources, load balancers, and possibly public IPv4 addresses. Review the configuration and current AWS pricing before running it.

```bash
kops update cluster "$CLUSTER_NAME" --yes --state "$KOPS_STATE_STORE"
```

### 5. Validate and connect

```bash
kops validate cluster "$CLUSTER_NAME" --state "$KOPS_STATE_STORE"
kubectl get nodes
kubectl get pods --all-namespaces
```

## Useful KOPS Commands

Preview infrastructure changes without applying them:

```bash
kops update cluster "$CLUSTER_NAME" --state "$KOPS_STATE_STORE"
```

Apply configuration changes to an existing cluster:

> **AWS COST WARNING:** Changes may create, replace, or delete billable AWS resources.

```bash
kops rolling-update cluster "$CLUSTER_NAME" --yes --state "$KOPS_STATE_STORE"
```

Delete the cluster when finished:

> **Important:** This terminates cluster infrastructure. Check the cluster and backups before running it.

```bash
kops delete cluster "$CLUSTER_NAME" --yes --state "$KOPS_STATE_STORE"
```

Remove the KOPS state bucket only after confirming that the cluster is deleted and the state is no longer needed:

> **AWS COST WARNING:** This permanently deletes objects in the state bucket.

```bash
aws s3 rb "$KOPS_STATE_STORE" --force
```

## Key Takeaways

- A distribution adds operational tooling and support around upstream Kubernetes.
- KOPS creates self-managed Kubernetes infrastructure on AWS; EKS is managed by AWS.
- The S3 state store is required by KOPS and should be protected and backed up.
- Route 53 must be correctly delegated when a custom cluster domain is used.
- `kops update cluster --yes` is the point where AWS infrastructure is created and costs can begin.
