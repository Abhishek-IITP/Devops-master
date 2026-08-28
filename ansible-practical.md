# Ansible Practical Notes

Ansible is an automation and configuration-management tool. It allows us to configure many remote servers from one machine instead of logging in to each server and running commands manually.

The machine from which Ansible commands are executed is called the **control node**. The servers that Ansible manages are called **managed nodes** or **target servers**.

```text
Control Node
		 |
		 | SSH
		 +------------------ Target Server 1
		 +------------------ Target Server 2
		 +------------------ Target Server 3
```

Ansible is agentless. We normally do not install an Ansible agent on the target servers. Ansible connects over SSH, runs the required module, and reports the result.

## 1. Prerequisites

Before using Ansible, make sure that:

- Ansible is installed on the control node.
- The target servers are running and reachable over the network.
- The control node can connect to the targets using SSH.
- The SSH user has permission to perform the required tasks.
- Python is available on the target servers, because most Ansible modules use Python remotely.
- The inventory contains the correct IP addresses or hostnames.

Check the Ansible installation:

```bash
ansible --version
```

## 2. SSH Access to Target Servers

Ansible uses SSH to communicate with Linux target servers. First, test a normal SSH connection from the control node:

```bash
ssh ubuntu@192.168.1.10
```

Replace `ubuntu` with the remote username and `192.168.1.10` with the target server's IP address or hostname.

### Password-Based SSH

Password authentication can be used for learning or for a quick initial connection. Ansible asks for the SSH password when the `-k` option is supplied:

```bash
ansible -i inventory.ini all -m ansible.builtin.ping -u ubuntu -k
```

Meaning of the important options:

- `-i inventory.ini`: use `inventory.ini` as the inventory file.
- `all`: run against every host in the selected inventory.
- `-m ansible.builtin.ping`: use Ansible's ping module to test connectivity.
- `-u ubuntu`: connect as the `ubuntu` user.
- `-k`: prompt for the SSH password.

If the remote user needs `sudo` privileges, use `-K` to ask for the privilege-escalation password:

```bash
ansible -i inventory.ini all -m ansible.builtin.ping -u ubuntu -k -K --become
```

Do not write real passwords directly in a playbook or commit them to Git. For practice, a password can be stored in inventory variables, but in a real project use Ansible Vault or another secret-management system.

### SSH Key-Based Authentication

SSH keys are the preferred method for regular Ansible automation because they avoid repeated password prompts and work well in CI/CD pipelines.

An SSH key pair contains two related files:

- **Private key:** stays on the control node. Never share it or commit it to Git.
- **Public key:** can be copied to the target server and stored in the user's `~/.ssh/authorized_keys` file.

Create a key pair on the control node:

```bash
ssh-keygen -t ed25519 -C "ansible-control-node"
```

Press Enter to accept the default path, or provide a different path when multiple keys are used. Copy the public key to the target server:

```bash
ssh-copy-id ubuntu@192.168.1.10
```

For cloud servers, the provider may give you a private key instead. Use it explicitly when testing the connection:

```bash
ssh -i ~/.ssh/aws-key.pem ubuntu@192.168.1.10
```

The private key must have restrictive permissions:

```bash
chmod 400 ~/.ssh/aws-key.pem
```

After key setup, verify that SSH works without a password prompt:

```bash
ssh ubuntu@192.168.1.10
```

## 3. Inventory File

The inventory tells Ansible which servers exist and how they are grouped. Create a file named `inventory.ini`:

```ini
[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[dbservers]
db1 ansible_host=192.168.1.12

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/aws-key.pem
```

Here, `web1`, `web2`, and `db1` are inventory names. `ansible_host` contains the actual address used for the SSH connection. The groups `webservers` and `dbservers` allow us to target different types of servers.

View the parsed inventory:

```bash
ansible-inventory -i inventory.ini --graph
ansible-inventory -i inventory.ini --list
```

Test all hosts before running a real configuration:

```bash
ansible -i inventory.ini all -m ansible.builtin.ping
```

Expected successful output contains `"ping": "pong"` for each reachable host.

## 4. Ansible Ad Hoc Commands

An ad hoc command is a single Ansible command used for a quick task. It is useful when we need an immediate answer or a one-time operation.

Ad hoc commands are not ideal for large, repeatable configuration because the steps are not organized in a file, reviewed easily, or documented as a complete workflow. Use a playbook when the work has multiple steps or needs to be repeated.

### Frequently Used Commands

Check connectivity:

```bash
ansible -i inventory.ini all -m ansible.builtin.ping
```

Display the uptime of every server:

```bash
ansible -i inventory.ini all -m ansible.builtin.command -a "uptime"
```

Run a command through a shell when shell features such as pipes or redirection are required:

```bash
ansible -i inventory.ini all -m ansible.builtin.shell -a "df -h | grep /dev"
```

Use `command` by default. Use `shell` only when shell syntax is genuinely needed, because shell commands can be less predictable and less secure.

Create a file:

```bash
ansible -i inventory.ini all -m ansible.builtin.file -a "path=/tmp/target state=touch"
```

Create a directory with specific permissions:

```bash
ansible -i inventory.ini webservers -m ansible.builtin.file -a "path=/opt/myapp state=directory mode=0755" --become
```

Install a package on Debian or Ubuntu:

```bash
ansible -i inventory.ini webservers -m ansible.builtin.apt -a "name=nginx state=present update_cache=true" --become
```

Install a package on RHEL, CentOS, or Fedora:

```bash
ansible -i inventory.ini webservers -m ansible.builtin.dnf -a "name=nginx state=present" --become
```

Start and enable a service:

```bash
ansible -i inventory.ini webservers -m ansible.builtin.service -a "name=nginx state=started enabled=true" --become
```

Gather system information:

```bash
ansible -i inventory.ini all -m ansible.builtin.setup
```

The `--become` option runs the task with elevated privileges, usually through `sudo`.

## 5. Inventory Groups and Host Targeting

Groups are useful when different servers have different responsibilities. For example:

- `webservers`: run web-server and reverse-proxy configuration.
- `dbservers`: install and configure database software.
- `kube_control_plane`: configure the Kubernetes control-plane node.
- `kube_workers`: configure the Kubernetes worker nodes.

If a playbook contains `hosts: webservers`, Ansible runs that play only on the hosts in the `webservers` group. It will not change the database servers or other groups.

You can also target hosts directly from the command line:

```bash
ansible -i inventory.ini webservers -m ansible.builtin.ping
ansible -i inventory.ini web1 -m ansible.builtin.ping
ansible -i inventory.ini 192.168.1.10 -m ansible.builtin.ping
```

Limit a playbook run to a group or a single server:

```bash
ansible-playbook -i inventory.ini site.yml --limit webservers
ansible-playbook -i inventory.ini site.yml --limit web1
```

You can select multiple targets with a comma-separated limit:

```bash
ansible-playbook -i inventory.ini site.yml --limit web1,web2
```

The `--limit` option is especially useful when testing a change on one server before applying it to the complete environment.

## 6. Writing a Playbook

A playbook is a YAML file containing one or more plays. A play connects a group of hosts to an ordered list of tasks.

Create `first-playbook.yml`:

```yaml
---
- name: Configure web servers
	hosts: webservers
	become: true

	tasks:
		- name: Install Nginx
			ansible.builtin.apt:
				name: nginx
				state: present
				update_cache: true

		- name: Ensure Nginx is running
			ansible.builtin.service:
				name: nginx
				state: started
				enabled: true
```

Important playbook concepts:

- `name`: a human-readable description shown in the output.
- `hosts`: the inventory group or host to configure.
- `become`: use elevated privileges for tasks that need them.
- `tasks`: the ordered list of operations.
- `ansible.builtin.apt`: the module used to manage an Ubuntu or Debian package.
- `state: present`: make sure the package exists; do not reinstall it unnecessarily.
- `state: started`: make sure the service is running.
- `enabled: true`: start the service automatically after a reboot.

Run the playbook:

```bash
ansible-playbook -i inventory.ini first-playbook.yml
```

Useful checks before or during a run: 

```bash
ansible-playbook -i inventory.ini first-playbook.yml --syntax-check
ansible-playbook -i inventory.ini first-playbook.yml --check
ansible-playbook -i inventory.ini first-playbook.yml --diff
ansible-playbook -i inventory.ini first-playbook.yml --limit web1
```

`--syntax-check` checks YAML and playbook structure. `--check` performs a dry run where supported. `--diff` displays file changes for modules that support it. These checks help catch mistakes before changing every server.

### Idempotency

An idempotent task produces the same final state when it is run multiple times. For example, this task ensures that Nginx is installed:

```yaml
- name: Install Nginx
	ansible.builtin.apt:
		name: nginx
		state: present
```

The first run may report `changed`, while later runs should report `ok` if nothing needs to change. Prefer Ansible modules with a desired state over repeatedly appending shell commands.

## 7. Variables, Templates, and Handlers

Variables prevent us from hard-coding values throughout a playbook:

```yaml
---
- name: Install an application package
	hosts: webservers
	become: true
	vars:
		package_name: nginx

	tasks:
		- name: Install the selected package
			ansible.builtin.apt:
				name: "{{ package_name }}"
				state: present
```

Templates are configuration files containing variables. They normally use the `.j2` extension and are rendered by Ansible with Jinja2.

Handlers are tasks that run only when another task reports a change. They are commonly used to restart a service after its configuration changes:

```yaml
tasks:
	- name: Deploy Nginx configuration
		ansible.builtin.template:
			src: nginx.conf.j2
			dest: /etc/nginx/nginx.conf
		notify: Restart Nginx

handlers:
	- name: Restart Nginx
		ansible.builtin.service:
			name: nginx
			state: restarted
```

## 8. Ansible Roles

As a playbook grows, keeping every task, template, variable, and handler in one file becomes difficult. A role provides a standard structure for reusable automation.

Create a role named `kubernetes`:

```bash
ansible-galaxy role init kubernetes
```

The generated role commonly contains:

```text
kubernetes/
├── defaults/main.yml     # low-priority default variables
├── handlers/main.yml    # handlers
├── tasks/main.yml       # main tasks
├── templates/           # Jinja2 templates
├── files/               # static files
├── vars/main.yml        # role variables
├── meta/main.yml        # role metadata and dependencies
└── README.md            # role documentation
```

A role is called from a playbook like this:

```yaml
---
- name: Configure Kubernetes nodes
	hosts: kube_control_plane:kube_workers
	become: true
	roles:
		- kubernetes
```

Roles make automation easier to reuse across development, staging, and production. They also make a project easier to test and maintain because each responsibility has a predictable location.

## 9. Practical Project: Kubernetes Cluster on AWS

The project combines Terraform and Ansible. Terraform creates the infrastructure, while Ansible configures the operating systems and Kubernetes software.

### Desired Architecture

```text`
AWS VPC
	|
	+-- EC2 instance 1: Kubernetes control plane
	+-- EC2 instance 2: Kubernetes worker 1
	+-- EC2 instance 3: Kubernetes worker 2
```

The control-plane node manages the cluster. Worker nodes run application workloads. In older tutorials, the control-plane node may be called the `master` node; `control plane` is the current terminology.

### Step 1: Create the EC2 Instances with Terraform

Use Terraform to create:

1. One EC2 instance for the Kubernetes control plane.
2. Two EC2 instances for Kubernetes workers.
3. A network that allows the nodes to communicate with each other.
4. Security-group rules for SSH and the Kubernetes ports required by the chosen installation method.
5. An output containing the public or private IP addresses needed by Ansible.

Do not hard-code changing EC2 IP addresses in the Ansible inventory. Generate an inventory from Terraform outputs, or update the inventory after the instances are created.

### Step 2: Create the Inventory

After Terraform creates the instances, organize them into Ansible groups:

```ini
[kube_control_plane]
control-plane ansible_host=CONTROL_PLANE_IP

[kube_workers]
worker1 ansible_host=WORKER_1_IP
worker2 ansible_host=WORKER_2_IP

[kubernetes:children]
kube_control_plane
kube_workers

[kubernetes:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/aws-key.pem
```

Replace the placeholder IP values with the actual addresses. Test all three nodes:

```bash
ansible -i inventory.ini kubernetes -m ansible.builtin.ping
```

### Step 3: Create the Kubernetes Playbook

Create the playbook file on the control node:

```bash
vi kubernetes.yml
```

A useful high-level playbook separates common node setup from control-plane and worker setup:

```yaml
---
- name: Prepare every Kubernetes node
	hosts: kubernetes
	become: true
	roles:
		- kubernetes

- name: Configure the Kubernetes control plane
	hosts: kube_control_plane
	become: true
	tasks:
		- name: Initialize the control plane
			ansible.builtin.command: kubeadm init --pod-network-cidr=10.244.0.0/16
			args:
				creates: /etc/kubernetes/admin.conf

- name: Configure Kubernetes workers
	hosts: kube_workers
	become: true
	tasks:
		- name: Join workers to the cluster
			ansible.builtin.command: "{{ kubeadm_join_command }}"
```

The exact Kubernetes tasks depend on the installation method, Kubernetes version, container runtime, and chosen network plugin. The playbook should normally handle tasks such as disabling swap, installing the container runtime, installing `kubeadm`, `kubelet`, and `kubectl`, initializing the control plane, installing a pod network, and joining the workers.

Do not copy a worker join token permanently into a public repository. Generate it on the control-plane node and pass it securely to the workers using Ansible variables or Vault.

### Step 4: Run the Project Safely

Validate the playbook first:

```bash
ansible-playbook -i inventory.ini kubernetes.yml --syntax-check
ansible-playbook -i inventory.ini kubernetes.yml --check
```

Run the common node setup:

```bash
ansible-playbook -i inventory.ini kubernetes.yml --limit kubernetes
```

When the nodes are ready, run the full playbook:

```bash
ansible-playbook -i inventory.ini kubernetes.yml
```

Verify the cluster from the control-plane node:

```bash
kubectl get nodes
kubectl get pods --all-namespaces
```

All three nodes should eventually appear in the expected state. Kubernetes installation may take time while the services start and the pod network becomes ready.

## 10. Recommended Learning Sequence

Follow this order while practicing:

1. Connect to one target server using normal SSH.
2. Configure SSH keys and test passwordless access.
3. Add the target to `inventory.ini`.
4. Run the Ansible ping module.
5. Practice ad hoc commands with safe modules such as `ping`, `file`, and `setup`.
6. Create a small playbook to install and start a package or service.
7. Practice groups and `--limit` with one server before using multiple servers.
8. Add variables, templates, and handlers.
9. Convert the playbook into a reusable role.
10. Use Terraform to create the three EC2 instances and Ansible to configure the Kubernetes cluster.
