# Ansible


# 🚀 Ansible 0 → Hero — Part 01

## Fundamentals + Architecture + Configuration

---

# 1. What is Ansible?

**Ansible is an open-source automation and configuration-management tool.**

It can automate:

* Linux server configuration
* Package installation
* Service management
* User creation
* Application deployment
* Server patching
* Rebooting
* Cloud infrastructure configuration
* Kubernetes node preparation
* Application configuration

The basic idea is:

```text
Without Ansible

Admin
 │
 ├── SSH → Server 1 → install package
 ├── SSH → Server 2 → install package
 ├── SSH → Server 3 → install package
 ├── SSH → Server 4 → restart service
 └── SSH → Server 5 → update OS
```

With Ansible:

```text
                    Ansible
                 Control Node
                      │
              ┌───────┼───────┐
              │       │       │
             SSH     SSH     SSH
              │       │       │
              ▼       ▼       ▼
           Server1 Server2 Server3
```


<img width="870" height="629" alt="image" src="https://github.com/user-attachments/assets/643ee720-56de-41ae-959c-1728b42c2b99" />


You define the desired operation once and Ansible executes it across your servers.

---

# 2. Why Ansible?

Imagine you have **100 Linux servers**.

You need to install:

```text
nginx
curl
git
vim
```

Manually:

```text
SSH → Server 1
SSH → Server 2
SSH → Server 3
...
SSH → Server 100
```

With Ansible:

```bash
ansible all -m apt -a "name=nginx state=present" --become
```

One command can manage the entire group.

---

# 3. Ansible Architecture

This is the most important concept.

```text
                    ┌─────────────────────┐
                    │   Ansible Control   │
                    │        Node         │
                    │                     │
                    │  Inventory         │
                    │  Playbooks          │
                    │  Variables          │
                    │  Modules            │
                    │  Roles              │
                    └──────────┬──────────┘
                               │
                              SSH
                 ┌─────────────┼─────────────┐
                 │             │             │
                 ▼             ▼             ▼
           ┌──────────┐  ┌──────────┐  ┌──────────┐
           │ Server 1 │  │ Server 2 │  │ Server 3 │
           │          │  │          │  │          │
           │ Linux    │  │ Linux    │  │ Linux    │
           └──────────┘  └──────────┘  └──────────┘
```

### Control Node

The machine where Ansible is installed.

Example:

```text
Ubuntu Laptop
Ubuntu VM
AWS EC2
Jenkins Agent
GitHub Actions Runner
```

The control node contains:

```text
ansible
ansible-playbook
inventory
playbooks
roles
configuration
```

---

# 4. Managed Nodes

These are the machines Ansible manages.

For example:

```text
web01
web02
db01
k8s-node01
k8s-node02
cnf-node01
```

Normally you **do not install Ansible itself** on these servers.

That's why Ansible is called:

> **Agentless automation**

---

# 5. How SSH Works

Ansible normally connects to Linux servers using SSH.

```text
Ansible Control Node
        │
        │ SSH
        ▼
    Server
        │
        ▼
Execute task
        │
        ▼
Return result
```

Example:

```bash
ssh ubuntu@192.168.1.10
```

If SSH works manually, Ansible can generally use the same connection.

---

# 6. SSH Key Authentication

For automation, SSH keys are preferred.

Create key:

```bash
ssh-keygen -t ed25519
```

You get:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

Think of them as:

```text
Private Key
    ↓
Stays on Control Node

Public Key
    ↓
Copied to Managed Server
```

Copy the key:

```bash
ssh-copy-id ubuntu@192.168.1.10
```

Test:

```bash
ssh ubuntu@192.168.1.10
```

You should be able to connect without entering the account password every time.

---

# 7. Install Ansible

Ubuntu/Debian:

```bash
sudo apt update
sudo apt install -y ansible
```

Verify:

```bash
ansible --version
```

Example:

```text
ansible [core 2.x.x]
python version = 3.x.x
```

---

# 8. Ansible Project Structure

Create:

```bash
mkdir ansible-zero-to-hero
cd ansible-zero-to-hero
```

Recommended structure:

```text
ansible-zero-to-hero/
│
├── ansible.cfg
│
├── inventory/
│   └── hosts.ini
│
├── group_vars/
│   └── all.yml
│
├── host_vars/
│
├── playbooks/
│   ├── install-packages.yml
│   ├── manage-services.yml
│   ├── update-servers.yml
│   └── reboot-servers.yml
│
├── roles/
│
└── README.md
```

---

# 9. ansible.cfg

This is the main Ansible configuration file.

```ini
[defaults]
inventory = inventory/hosts.ini
host_key_checking = False
retry_files_enabled = False
interpreter_python = auto_silent
timeout = 30

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False
```

Now Ansible automatically knows:

```text
Inventory:
inventory/hosts.ini
```

So instead of:

```bash
ansible all -i inventory/hosts.ini -m ping
```

you can simply use:

```bash
ansible all -m ping
```

### Important

For production, don't blindly disable SSH host-key checking. Understand the security implications first.

---

# 10. Inventory

Inventory tells Ansible:

> **Which servers should I manage?**

Example:

```ini
[webservers]
web01 ansible_host=192.168.1.101
web02 ansible_host=192.168.1.102

[dbservers]
db01 ansible_host=192.168.1.103

[k8s_nodes]
k8s01 ansible_host=192.168.1.104
k8s02 ansible_host=192.168.1.105
```

Now Ansible knows:

```text
webservers
 ├── web01
 └── web02

dbservers
 └── db01

k8s_nodes
 ├── k8s01
 └── k8s02
```

---

# 11. Inventory Variables

You can specify the SSH user:

```ini
[webservers]
web01 ansible_host=192.168.1.101
web02 ansible_host=192.168.1.102

[webservers:vars]
ansible_user=ubuntu
```

You can also specify an SSH private key:

```ini
[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# 12. Test Connectivity

First:

```bash
ansible all -m ping
```

Expected:

```text
web01 | SUCCESS => {
    "ping": "pong"
}

web02 | SUCCESS => {
    "ping": "pong"
}
```

This is your first important Ansible test.

Remember:

```text
ansible
   ↓
Inventory
   ↓
SSH
   ↓
Managed Node
   ↓
Module
   ↓
Result
```

---

# 13. What is a Module?

A module performs an action.

Examples:

```text
apt
yum
dnf
service
systemd
copy
file
user
template
command
shell
reboot
package
```

For example:

```bash
ansible webservers -m ansible.builtin.apt \
  -a "name=nginx state=present" \
  --become
```

Meaning:

```text
webservers
     ↓
apt module
     ↓
Install nginx
     ↓
sudo
```

---

# 14. Ad-Hoc Commands

Ad-hoc means:

> Run a quick operation without creating a playbook.

Check uptime:

```bash
ansible all -a "uptime"
```

Check disk:

```bash
ansible all -a "df -h"
```

Check memory:

```bash
ansible all -a "free -m"
```

Check OS:

```bash
ansible all -a "cat /etc/os-release"
```

---

# 15. What is a Playbook?

When automation becomes more complex, don't use ad-hoc commands.

Create a **Playbook**.

Example:

```yaml
---
- name: Install Nginx
  hosts: webservers
  become: true

  tasks:

    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

Run:

```bash
ansible-playbook playbooks/install-nginx.yml
```

---

# 16. Understand the Playbook Structure

This:

```yaml
- name: Install Nginx
```

is the **Play**.

This:

```yaml
hosts: webservers
```

means:

> Run this play against the `webservers` inventory group.

This:

```yaml
become: true
```

means:

> Use privilege escalation, normally sudo.

And:

```yaml
tasks:
```

contains the actions.

Then:

```yaml
- name: Install nginx
  ansible.builtin.apt:
```

is a task using the **apt module**.

---

# 17. Ansible Execution Flow

This is the mental model I want you to remember:

```text
                 ansible-playbook
                        │
                        ▼
                    Playbook
                        │
                        ▼
                    Inventory
                        │
                        ▼
                      Hosts
                        │
                        ▼
                       SSH
                        │
                        ▼
                     Module
                        │
                        ▼
                  Remote Server
                        │
                        ▼
                     Result
                        │
                        ▼
               changed / ok / failed
```

---

# 18. The Ansible Configuration Hierarchy

As you progress, you'll encounter:

```text
ansible.cfg
     │
     ▼
Inventory
     │
     ▼
Variables
     │
     ▼
Playbook
     │
     ▼
Tasks
     │
     ▼
Modules
     │
     ▼
Handlers
     │
     ▼
Roles
```

And eventually:

```text
Terraform
     ↓
Infrastructure
     ↓
Ansible
     ↓
OS configuration
     ↓
Kubernetes
     ↓
CNF
     ↓
5G Core
```

---

# 19. Terraform vs Ansible

Since we're already learning Terraform, remember this distinction:

### Terraform

```text
"What infrastructure should exist?"
```

Example:

```text
VPC
EC2
EBS
Load Balancer
Security Group
```

### Ansible

```text
"How should that server be configured?"
```

Example:

```text
Install packages
Configure Linux
Create users
Deploy config
Start services
Patch server
Reboot server
```

### Together

```text
Terraform
    ↓
Create EC2
    ↓
Ansible
    ↓
Configure EC2
    ↓
Install Docker
    ↓
Configure Kubernetes
    ↓
Deploy application
```

---

# 20. Your Ansible Learning Journey



```text
01 Fundamentals
      ↓
02 Architecture
      ↓
03 Installation
      ↓
04 SSH
      ↓
05 Inventory
      ↓
06 Ad-hoc Commands
      ↓
07 Playbooks
      ↓
08 Modules
      ↓
09 Variables
      ↓
10 Facts
      ↓
11 Conditions
      ↓
12 Loops
      ↓
13 Templates
      ↓
14 Handlers
      ↓
15 Server Management
      ↓
16 Package Installation
      ↓
17 Service Management
      ↓
18 Server Patching
      ↓
19 Reboot
      ↓
20 Roles
      ↓
21 Vault
      ↓
22 AWS
      ↓
23 Terraform + Ansible
      ↓
24 Kubernetes
      ↓
25 CNF / 5G Automation
      ↓
26 CI/CD
      ↓
27 Real-world Projects
```

### 🔥 The key idea

Don't memorize Ansible commands.

Understand this:

```text
Inventory = WHERE
Playbook = WHAT
Module = HOW
Variables = VALUES
Conditionals = WHEN
Loops = REPEAT
Handlers = REACT
Roles = ORGANIZE
Vault = PROTECT SECRETS
```

Once these concepts are clear, Ansible becomes much easier.
