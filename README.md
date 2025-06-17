# Ansible Project To Monitor VMs Health

A comprehensive Ansible project for monitoring the health of AWS EC2 instances with automated tagging and dynamic inventory management.

## Prerequisites

- Ubuntu/Debian-based system
- AWS Account with appropriate permissions
- SSH key pair for EC2 access

## Installation & Setup

### Step 1: Update System
```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2: Install Ansible
Add the official Ansible PPA and install:
```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

### Step 3: Install AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

### Step 4: Setup Python Environment
```bash
# Install venv module
sudo apt install python3-venv -y

# Create and activate virtual environment
python3 -m venv ansible-env
source ansible-env/bin/activate

# Install required Python packages
pip install boto3 botocore docker

# Install Ansible AWS collection
ansible-galaxy collection install amazon.aws
```

## Configuration

### Ansible Configuration (ansible.cfg)
```ini
[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

### Dynamic Inventory (inventory/aws_ec2.yaml)
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
  - key: tags.Name
    prefix: name
  - key: tags.Environment
    prefix: env
```

## Scripts

### EC2 Instance Tagging Script
This script automatically tags your EC2 instances with sequential names:

```bash
#!/bin/bash

# Fetch instance IDs that match Environment=dev and Role=web
instance_ids=$(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=dev" "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

# Sort instance IDs deterministically
sorted_ids=($(echo "$instance_ids" | tr '\t' '\n' | sort))

# Rename instances sequentially
counter=1
for id in "${sorted_ids[@]}"; do
  name="web-$(printf "%02d" $counter)"
  echo "Tagging $id as $name"
  aws ec2 create-tags --resources "$id" \
    --tags Key=Name,Value="$name"
  ((counter++))
done
```

### SSH Key Distribution Script
This script copies your public SSH key to all EC2 instances:

```bash
#!/bin/bash

# Define variables
PEM_FILE="DevOps-Shack.pem"
PUB_KEY=$(cat ~/.ssh/id_rsa.pub)
USER="ubuntu"  # or ec2-user
INVENTORY_FILE="inventory/aws_ec2.yaml"

# Extract hostnames/IPs from dynamic inventory
HOSTS=$(ansible-inventory -i $INVENTORY_FILE --list | jq -r '._meta.hostvars | keys[]')

for HOST in $HOSTS; do
  echo "Injecting key into $HOST"
  ssh -o StrictHostKeyChecking=no -i $PEM_FILE $USER@$HOST "
    mkdir -p ~/.ssh && \
    echo \"$PUB_KEY\" >> ~/.ssh/authorized_keys && \
    chmod 700 ~/.ssh && \
    chmod 600 ~/.ssh/authorized_keys
  "
done
```

## Usage

### 1. Clone the Project
```bash
git clone https://github.com/jaiswaladi246/Ansible-VM-Monitor.git
cd Ansible-VM-Monitor
```

### 2. Verify Dynamic Inventory
```bash
ansible-inventory -i inventory/aws_ec2.yaml --graph
```

### 3. Run the Monitoring Playbook
```bash
ansible-playbook playbook.yaml
```

## Project Structure
```
Ansible-VM-Monitor/
├── ansible.cfg
├── inventory/
│   └── aws_ec2.yaml
├── playbook.yaml
├── scripts/
│   ├── tag_instances.sh
│   └── copy_ssh_key.sh
└── README.md
```

## Important Notes

- Ensure your AWS credentials are properly configured
- Make sure your EC2 instances have the appropriate tags (Environment=dev)
- The PEM file path should be updated according to your setup
- All instances should be in the 'running' state for the inventory to detect them

## Troubleshooting

- If you encounter SSH connection issues, verify that your security groups allow SSH access
- Ensure the correct user (ubuntu/ec2-user) is used based on your AMI
- Check that your AWS credentials have the necessary EC2 permissions

## Repository
GitHub: https://github.com/jaiswaladi246/Ansible-VM-Monitor.git