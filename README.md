# VPS Setup Playbook

This Ansible playbook automates the setup and configuration of a VPS.

## Prerequisites

- Ansible installed on your local machine
- SSH access to your VPS

## Installing Ansible

### On Ubuntu/Debian:

```bash
# Add the Ansible PPA repository
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# Verify installation
ansible --version
```

### On CentOS/RHEL:

```bash
# Install EPEL repository
sudo yum install epel-release
sudo yum install ansible

# Verify installation
ansible --version
```

### Using pip (works on most systems):

```bash
# Install pip if needed
sudo apt install python3-pip  # Debian/Ubuntu
# OR
sudo yum install python3-pip  # CentOS/RHEL

# Install Ansible
pip3 install ansible

# Verify installation
ansible --version
```

### On macOS:

```bash
# Using Homebrew
brew install ansible

# Verify installation
ansible --version
```

## Usage

1. Update the inventory file with your VPS details:

   ```
   # Edit inventory.ini
   [vps]
   your_vps ansible_host=YOUR_VPS_IP ansible_user=root
   ```

2. Customize variables in `group_vars/all.yml` to match your requirements.

3. Run the playbook:

   ```bash
   ansible-playbook -i inventory.ini vps-setup.yml
   ```

4. If SSH password is required:

   ```bash
   ansible-playbook -i inventory.ini vps-setup.yml --ask-pass
   ```

5. If sudo password is required:

   ```bash
   ansible-playbook -i inventory.ini vps-setup.yml --ask-become-pass
   ```

## Features

- System updates and package installation
- User management
- Basic security configuration with UFW and fail2ban
- Timezone and hostname configuration

## Important Notes

- The playbook will set up an admin user and disable password authentication
- Make sure your SSH key is properly set up before disabling password authentication
- You need to set the `admin_password` variable in a secure way (consider using Ansible Vault)
