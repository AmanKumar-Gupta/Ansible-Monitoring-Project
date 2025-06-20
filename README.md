# Ansible AWS EC2 VM Health Monitoring Project

A robust Ansible-based solution for monitoring the health of AWS EC2 instances, featuring automated tagging, dynamic inventory, and consolidated email reporting.

---

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Configuration](#configuration)
- [Scripts](#scripts)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Features

- **Dynamic AWS EC2 Inventory** using Ansible's AWS plugin
- **Automated Tagging** of EC2 instances for easy identification
- **Health Metrics Collection** (CPU, Memory, Disk) from all running instances
- **Animated HTML Email Reports** with consolidated VM health data
- **Easy SSH Key Distribution** to all managed instances

---

## Prerequisites

- Ubuntu/Debian-based system
- AWS account with EC2 permissions
- SSH key pair for EC2 access
- Python 3.x

---

## Installation & Setup

### 1. System Update

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install Ansible

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### 3. Install AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

### 4. Python Virtual Environment & Dependencies

```bash
sudo apt install python3-venv -y
python3 -m venv ansible-env
source ansible-env/bin/activate
pip install boto3 botocore docker
ansible-galaxy collection install amazon.aws
```

---

## Configuration

### Ansible Configuration ([ansible.cfg](ansible.cfg))

```ini
[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False
remote_user = ubuntu
gathering = smart

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

### Dynamic Inventory ([inventory/aws_ec2.yaml](inventory/aws_ec2.yaml))

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: dev
  instance-state-name: running
compose:
  ansible_host: public_ip_address
keyed_groups:
  - key: tags.Environment
    prefix: env
```

### Group Variables ([group_vars/all.yaml](group_vars/all.yaml))

Set your SMTP and email credentials for reporting:

```yaml
smtp_server: "smtp.gmail.com"
smtp_port: 587
email_user: "<your_email>"
email_pass: "<your_app_password>"
alert_recipient: "<recipient_email>"
```

---

## Scripts

### 1. EC2 Instance Tagging ([tagging-script.sh](tagging-script.sh))

Automatically tags EC2 instances with sequential names.

### 2. SSH Key Distribution ([copy-public-key.sh](copy-public-key.sh))

Copies your public SSH key to all EC2 instances in the dynamic inventory.

---

## Usage

### 1. Clone the Repository

```bash
git clone https://github.com/jaiswaladi246/Ansible-VM-Monitor.git
cd Ansible-VM-Monitor
```

### 2. Verify Dynamic Inventory

```bash
ansible-inventory -i inventory/aws_ec2.yaml --graph
```

### 3. Tag EC2 Instances

```bash
bash tagging-script.sh
```

### 4. Distribute SSH Key

```bash
bash copy-public-key.sh
```

### 5. Run the Monitoring Playbook

```bash
ansible-playbook playbook.yaml
```

---

## Project Structure

```
Ansible-VM-Monitor/
├── ansible.cfg
├── collect_metrics.yaml
├── copy-public-key.sh
├── group_vars/
│   └── all.yaml
├── inventory/
│   └── aws_ec2.yaml
├── playbook.yaml
├── README.md
├── send_report.yaml
├── tagging-script.sh
├── templates/
│   └── report_email_animated.html.j2
└── Screenshots/
    ├── Instances.png
    ├── successful.png
    └── tag-script.png
```

---

## Troubleshooting

- **SSH Issues:** Ensure security groups allow SSH and the correct user (ubuntu/ec2-user) is used.
- **AWS Credentials:** Confirm your AWS credentials are configured and have necessary permissions.
- **Inventory Issues:** Make sure EC2 instances are tagged correctly and in the 'running' state.
- **Email Sending:** Use an app password for Gmail or enable SMTP for your email provider.

---

## License

This project is licensed under the MIT License.

---
