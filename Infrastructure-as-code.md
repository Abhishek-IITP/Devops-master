# Infrastructure as Code and Terraform

## 1. What Is Infrastructure?

Infrastructure includes the computing resources required to run an application. Examples include:

- Virtual machines and containers
- Networks, subnets, and routing tables
- Firewalls and security groups
- Load balancers and DNS records
- Databases and storage buckets
- Monitoring, logging, and identity permissions

Traditionally, engineers created these resources manually through a cloud console or by running provider-specific scripts. That approach works for a small experiment, but it becomes difficult to repeat, review, and maintain as an organization grows.

## 2. What Is Infrastructure as Code?

**Infrastructure as Code (IaC)** means defining infrastructure in machine-readable configuration files and managing those files like application source code.

Instead of clicking through a console to create a server, we describe the desired result in code. An IaC tool then compares the desired configuration with the real environment and creates, updates, or removes resources as necessary.

```text
Configuration Files
				|
				v
IaC Tool compares desired state with real state
				|
				v
Cloud or data-center APIs
				|
				v
Provisioned Infrastructure
```

IaC files can be stored in Git, reviewed through pull requests, tested in CI/CD, and used to recreate an environment consistently.

## 3. Why Do We Need Infrastructure Management?

Infrastructure management is needed to make environments:

- **Consistent:** development, staging, and production can follow the same design.
- **Repeatable:** the same configuration can be created again without repeating manual steps.
- **Reviewable:** changes can be inspected before they are applied.
- **Auditable:** Git history shows who changed the infrastructure and why.
- **Scalable:** one configuration can manage a few resources or many resources.
- **Recoverable:** infrastructure can be recreated after a failure.
- **Less error-prone:** the tool performs predictable operations instead of relying on manual clicks.

IaC does not remove the need for design decisions. It makes those decisions explicit and automates their implementation.

## 4. Scenario 1: Moving from AWS to Azure or On-Premises

Imagine that a company initially uses AWS. A DevOps engineer writes many CloudFormation Templates (CFTs) to automate AWS infrastructure.

Later, the company decides to move to Azure. CloudFormation is an AWS-specific service, so the existing templates cannot directly create Azure resources. The team must translate or rewrite the templates using Azure Resource Manager templates, Bicep, scripts, or another Azure-specific tool.

After some time, the company decides to build a private cloud on its own servers using OpenStack. The Azure definitions also cannot directly create OpenStack resources. The team now has to learn another platform and create another set of automation files.

```text
AWS + CloudFormation
					|
					v
Azure + ARM/Bicep
					|
					v
OpenStack + OpenStack tooling
```

### Problems with the Provider-Specific Approach

- The team maintains different languages, tools, and file formats.
- Engineers repeat the same design in a new syntax during every migration.
- Knowledge becomes tightly coupled to one provider.
- Testing and code review must account for multiple toolchains.
- Migration work can consume time that should be spent improving the platform.

### How Terraform Helps

Terraform provides one declarative workflow for many providers. It uses provider plugins to translate Terraform resources into calls to AWS, Azure, OpenStack, Kubernetes, and other APIs.

For example, the overall workflow remains similar:

```bash
terraform init
terraform plan
terraform apply
```

The provider changes, and the resource definitions may need some changes, but the team continues to use the same core workflow, state model, dependency graph, and review process.

This reduces migration effort because reusable concepts such as networking, naming, tagging, variables, outputs, modules, and environment structure can be carried forward.

### Important Limitation

Terraform is not a magic conversion tool. AWS, Azure, and OpenStack expose different capabilities, resource models, security systems, and pricing structures. A load balancer or managed database on one provider may not have an exact equivalent on another provider.

Therefore, Terraform improves portability of the **workflow and automation approach**, but it does not guarantee that one configuration can run unchanged everywhere. Provider-specific knowledge is still required.

## 5. Scenario 2: A Hybrid-Cloud Company

Many companies use more than one platform. For example:

- AWS provides object storage and analytics.
- Azure hosts internal DevOps tools or identity services.
- A private data center runs systems that cannot be moved to the public cloud.

Without a common IaC tool, engineers may use CloudFormation for AWS, Bicep for Azure, and separate scripts or OpenStack Heat for the private cloud. This creates several different workflows.

With Terraform, the team can keep a similar project structure and execution process:

```text
Terraform configuration
			 |
			 +-- AWS provider  ------> AWS resources
			 +-- Azure provider -----> Azure resources
			 +-- OpenStack provider -> Private-cloud resources
```

Terraform can also express dependencies across resources. For example, an application deployment may depend on a network, identity permissions, a storage bucket, and a database being available first.

### What Terraform Provides in Hybrid Cloud

- A common configuration language: HCL.
- A common command workflow: `init`, `validate`, `plan`, and `apply`.
- A dependency graph that determines resource creation order.
- A state file that records resources Terraform manages.
- Modules for reusable infrastructure patterns.
- Variables and outputs for environment-specific values.
- A reviewable plan before changes are applied.
- Integration with Git, CI/CD, and policy checks.

### What Terraform Does Not Provide

- It does not make all clouds identical.
- It does not automatically design a secure network.
- It does not remove provider quotas, pricing differences, or API limitations.
- It does not replace application deployment tools such as Ansible, Helm, or Kubernetes controllers.
- It does not make secrets safe if they are placed directly in configuration or state files.

Terraform reduces the number of tools and workflows the team must learn, but engineers still need to understand the resources they are creating.

## 6. Terraform Core Concepts

### Providers

A provider is a plugin that allows Terraform to communicate with a platform or service API.

```hcl
terraform {
	required_providers {
		aws = {
			source  = "hashicorp/aws"
			version = "~> 6.0"
		}
	}
}

provider "aws" {
	region = "us-east-1"
}
```

The version constraint helps keep provider upgrades controlled and predictable.

### Resources

A resource describes an infrastructure object that Terraform should create or manage.

```hcl
resource "aws_s3_bucket" "logs" {
	bucket = "company-example-logs"
}
```

The resource type is `aws_s3_bucket`, and the local Terraform name is `logs`.

### Variables

Variables allow the same configuration to work in different environments:

```hcl
variable "environment" {
	type    = string
	default = "dev"
}
```

### Outputs

Outputs expose useful values after Terraform creates resources:

```hcl
output "log_bucket_name" {
	value = aws_s3_bucket.logs.bucket
}
```

### Modules

A module is a reusable collection of Terraform configuration. For example, a networking module can create a VPC, subnets, routes, and security groups using a consistent pattern.

### State

Terraform state records the relationship between configuration and real resources. Terraform uses it to determine what already exists and what must change.

For team use, store state in a secure remote backend with locking. Do not casually commit a state file to a public repository because it may contain sensitive values or infrastructure details.

## 7. Standard Terraform Workflow

### Step 1: Write Configuration

Create `.tf` files describing the desired infrastructure.

### Step 2: Initialize

Download providers and initialize the working directory:

```bash
terraform init
```

### Step 3: Format and Validate

```bash
terraform fmt
terraform validate
```

`fmt` applies standard Terraform formatting. `validate` checks whether the configuration is syntactically valid and internally consistent.

### Step 4: Review the Plan

```bash
terraform plan
```

The plan shows proposed additions, changes, and deletions. Review it carefully, especially before production changes.

### Step 5: Apply the Plan

```bash
terraform apply
```

Terraform asks for confirmation unless automatic approval is configured. In CI/CD, it is often safer to save a reviewed plan and apply that exact plan.

### Step 6: Destroy Temporary Resources

Only for resources that are no longer needed:

```bash
terraform destroy
```

Never run `destroy` in a shared or production environment without understanding exactly what will be removed.

## 8. Infrastructure as Code vs API as Code

The phrase **API as code** can be confusing because Terraform itself uses provider APIs. The two ideas are related, but they are not exactly the same.

### Direct API or Imperative Script

An engineer directly calls an API or writes a script that says what actions to perform:

```text
1. Call the API to create a network.
2. Call the API to create a subnet.
3. Call the API to create a server.
4. Save the returned IDs.
5. Write separate logic for updates and cleanup.
```

This approach focuses on **how** to perform each operation. The script must usually handle ordering, retries, existing resources, failures, and deletion logic.

### Declarative Infrastructure as Code

Terraform describes the desired end state:

```text
The network, subnet, and server should exist with these properties.
```

Terraform compares the desired state with its state and the provider APIs, creates a dependency graph, and calculates the required actions.

### Why Terraform Is Often Better Than Direct API Scripts

- **Desired-state model:** describe the result instead of manually coding every API call.
- **Plan before execution:** inspect proposed changes before applying them.
- **Dependency handling:** Terraform understands relationships between resources.
- **Idempotent behavior:** repeated runs converge toward the same desired state.
- **State tracking:** Terraform remembers resources it manages.
- **Reusable modules:** common patterns can be packaged and shared.
- **Provider abstraction:** different platforms use a consistent workflow.
- **Git and CI/CD support:** configuration can be reviewed, tested, and automated.
- **Dependency-aware deletion:** Terraform can remove resources in a controlled order.

### When Direct API Code Is Still Useful

Direct API calls are appropriate when:

- A provider feature is not yet supported by Terraform.
- The operation is a one-time data or service action rather than infrastructure management.
- Complex custom business logic is required.
- A platform requires a custom controller or event-driven workflow.

Terraform is best understood as declarative infrastructure code that uses APIs underneath. It is not accurate to say that API as code completely replaces IaC; in many cases, Terraform is the safer abstraction layer over infrastructure APIs.

## 9. Interview Scenario Questions and Answers

### Scenario 1: Cloud Migration

**Question:** Your company has 100 AWS CloudFormation templates and decides to move to Azure. How would you reduce the migration effort?

**Answer:** I would first inventory the existing architecture and separate provider-independent design from AWS-specific implementation. I would evaluate Terraform because it supports both AWS and Azure through providers. I would convert common patterns into Terraform modules and replace AWS-specific resources with Azure equivalents. I would validate each module in a non-production environment and migrate incrementally. I would not assume that every resource has a direct one-to-one mapping.

### Scenario 2: Hybrid Cloud

**Question:** The company uses AWS for storage and Azure for DevOps services. How can Terraform help?

**Answer:** Terraform can use multiple providers in one project or in separate modules. It gives the team a consistent HCL configuration style, plan/apply workflow, state management approach, and CI/CD integration. AWS and Azure resources still need provider-specific definitions, but Terraform reduces toolchain duplication and makes dependencies and changes reviewable.

### Scenario 3: Unexpected Destruction

**Question:** A Terraform plan shows that a production database will be destroyed. What would you do?

**Answer:** I would not apply the plan. I would inspect the configuration and state, check whether the resource address or immutable arguments changed, and compare the change with the Git history. I would run `terraform plan` with the correct workspace and variables, verify the remote backend, and discuss the change with the owner. I would use lifecycle protections such as `prevent_destroy` where appropriate, but I would not use them as a substitute for understanding the root cause.

### Scenario 4: State Locking

**Question:** Two engineers run `terraform apply` at the same time. How do you prevent state corruption?

**Answer:** I would use a shared remote backend that supports state locking. The lock prevents concurrent writes. I would also run Terraform through a controlled CI/CD process, limit production permissions, and never delete a lock manually unless I have confirmed that no Terraform process is still running.

### Scenario 5: Drift

**Question:** Someone changes a security-group rule manually in the cloud console. How would Terraform handle it?

**Answer:** During refresh and planning, Terraform can detect differences between the configuration, state, and real infrastructure. The plan may propose restoring the rule to the configuration-defined state. I would investigate why the manual change happened, update the code if the change is legitimate, or apply the reviewed plan to bring the environment back to the declared state.

### Scenario 6: Secrets

**Question:** How would you manage database passwords in Terraform?

**Answer:** I would avoid hard-coding secrets in `.tf` files or committing variable files containing plaintext credentials. I would use a secrets manager, CI/CD secret variables, or an encrypted Terraform mechanism such as a carefully managed Vault workflow. I would also secure the remote state backend because sensitive values can appear in state even when they are marked sensitive in output.

### Scenario 7: Terraform and Ansible

**Question:** When would you use Terraform and when would you use Ansible?

**Answer:** I would generally use Terraform to provision infrastructure resources such as networks, instances, load balancers, and databases. I would use Ansible to configure operating systems, install packages, deploy configuration files, and manage services on existing servers. Terraform creates the platform; Ansible configures the machine and application layer. The tools can be used together, but their responsibilities should remain clear.

### Scenario 8: Reusable Design

**Question:** Three teams need the same network pattern in development, staging, and production. How would you avoid duplication?

**Answer:** I would create a reusable Terraform module with variables for CIDR ranges, environment names, tags, and optional features. Each environment would call the module with its own values. I would version the module, test it, and review changes before upgrading environments.

### Scenario 9: Terraform Portability

**Question:** Can you use the exact same Terraform code for AWS, Azure, and OpenStack?

**Answer:** Usually not. Terraform provides a common language and workflow, but providers expose different resource types and arguments. I would reuse the architecture principles, module conventions, naming, tagging, variable patterns, and CI/CD process, while implementing provider-specific modules where necessary.

### Scenario 10: Failed Apply

**Question:** `terraform apply` fails halfway through. What is your approach?

**Answer:** I would read the error carefully, inspect the resulting plan, and verify which resources were created successfully. I would fix the underlying issue, such as permissions, quotas, dependency problems, or invalid arguments, and run the plan again. I would avoid manually deleting resources or editing state unless the situation requires it and the consequences are understood.

## 10. Key Takeaways

- Infrastructure as Code defines infrastructure in version-controlled configuration files.
- Terraform provides a common declarative workflow across many providers.
- Terraform improves portability, but cloud-specific knowledge is still required.
- State, remote backends, locking, and secret handling are critical in team environments.
- Use `plan` to review changes before `apply`.
- Use modules to share repeatable infrastructure patterns.
- Terraform is generally for provisioning infrastructure; Ansible is commonly used for server configuration.
- Direct API scripts still have valid use cases, but Terraform handles desired state, dependencies, planning, and lifecycle management for infrastructure more effectively.