# fail2ban Management Guide

## Table of Contents

1. [Overview](#overview)
2. [Current Configuration](#current-configuration)
3. [Common Management Tasks](#common-management-tasks)
4. [Using the Ansible Management Playbook](#using-the-ansible-management-playbook)
5. [Manual Command Line Management](#manual-command-line-management)
6. [Troubleshooting](#troubleshooting)
7. [Log Analysis](#log-analysis)
8. [Best Practices](#best-practices)

## Overview

fail2ban is an intrusion prevention system that monitors log files and automatically bans IP addresses that show signs of malicious activity, such as repeated failed login attempts.

## Current Configuration

Your fail2ban setup includes:

- **Ban Time**: 1 hour (`1h`)
- **Find Time**: 15 minutes (`15m`) - the time window to count failures
- **Max Retry**: 5 attempts for general services
- **SSH Max Retry**: 5 attempts for SSH specifically
- **Ban Action**: UFW (Uncomplicated Firewall)
- **No Ignored IPs**: All IPs are subject to the same rules

### Protected Services

- **SSH (sshd)**: Monitors `/var/log/auth.log` for failed login attempts

## Common Management Tasks

### 1. Check fail2ban Status

#### Using Ansible Playbook:

```bash
cd /path/to/PFE-PLAYBOOK
ansible-playbook -i inventory.ini fail2ban-manage.yml
# Choose 'status' when prompted
```

#### Manual Command:

```bash
# SSH into your server first
ssh admin@your-server-ip

# Check general status
sudo fail2ban-client status

# Check specific jail status
sudo fail2ban-client status sshd
```

### 2. Unban an IP Address

#### Using Ansible Playbook:

```bash
cd /path/to/PFE-PLAYBOOK
ansible-playbook -i inventory.ini fail2ban-manage.yml
# Choose 'unban' when prompted
# Enter the IP address to unban
```

#### Manual Command:

```bash
# Replace YOUR_IP with the actual IP address
sudo fail2ban-client set sshd unbanip YOUR_IP
```

### 3. Restart fail2ban Service

#### Using Ansible Playbook:

```bash
cd /path/to/PFE-PLAYBOOK
ansible-playbook -i inventory.ini fail2ban-manage.yml
# Choose 'restart' when prompted
```

#### Manual Commands:

```bash
# Restart the service
sudo systemctl restart fail2ban

# Or reload configuration without restarting
sudo fail2ban-client reload
```

## Using the Ansible Management Playbook

The `fail2ban-manage.yml` playbook provides an interactive way to manage fail2ban remotely:

### Available Actions:

1. **status** - Check current fail2ban status and banned IPs
2. **unban** - Unban a specific IP address
3. **restart** - Restart the fail2ban service

### Usage Examples:

```bash
# Navigate to your playbook directory
cd /home/josemokeni/PFE-PLAYBOOK

# Run the management playbook
ansible-playbook -i inventory.ini fail2ban-manage.yml

# You'll be prompted to choose an action:
# What action do you want to perform? (status/unban/restart) [status]:

# For unban action, you'll also be prompted for the IP:
# IP address to unban (leave empty if not unbanning) []:
```

## Manual Command Line Management

When directly connected to your server, you can use these commands:

### Basic Status Commands

```bash
# General fail2ban status
sudo fail2ban-client status

# Detailed status of SSH jail
sudo fail2ban-client status sshd

# List all active jails
sudo fail2ban-client status | grep "Jail list"
```

### Ban Management Commands

```bash
# Unban a specific IP from SSH jail
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Ban an IP manually (if needed)
sudo fail2ban-client set sshd banip 192.168.1.100

# Get list of banned IPs for SSH
sudo fail2ban-client get sshd banip
```

### Service Management

```bash
# Reload configuration
sudo fail2ban-client reload

# Reload specific jail
sudo fail2ban-client reload sshd

# Stop fail2ban
sudo fail2ban-client stop

# Start fail2ban
sudo fail2ban-client start

# Restart via systemctl
sudo systemctl restart fail2ban
sudo systemctl status fail2ban
```

## Troubleshooting

### Common Issues and Solutions

#### 1. You've Been Banned

**Symptoms**: Cannot SSH to your server
**Solution**:

```bash
# From another machine or using the console:
sudo fail2ban-client set sshd unbanip YOUR_IP_ADDRESS

# Or restart fail2ban to clear all bans:
sudo systemctl restart fail2ban
```

#### 2. fail2ban Not Starting

**Check the service status**:

```bash
sudo systemctl status fail2ban
sudo journalctl -u fail2ban -f
```

**Common fixes**:

```bash
# Check configuration syntax
sudo fail2ban-client -t

# Fix file permissions
sudo chown root:root /etc/fail2ban/jail.local
sudo chmod 644 /etc/fail2ban/jail.local
```

#### 3. Configuration Not Applied

```bash
# Reload after making changes
sudo fail2ban-client reload

# Or restart the service
sudo systemctl restart fail2ban
```

### Emergency Access

If you're locked out and need emergency access:

1. **Use server console** (if available through your hosting provider)
2. **Contact your hosting provider** to whitelist your IP
3. **Use a different IP address** (mobile hotspot, VPN, etc.)

## Log Analysis

### View fail2ban Logs

```bash
# Recent fail2ban activity
sudo tail -f /var/log/fail2ban.log

# Search for specific IP
sudo grep "192.168.1.100" /var/log/fail2ban.log

# View SSH authentication attempts
sudo tail -f /var/log/auth.log
```

### Useful Log Patterns

```bash
# See recent bans
sudo grep "Ban" /var/log/fail2ban.log | tail -10

# See recent unbans
sudo grep "Unban" /var/log/fail2ban.log | tail -10

# Monitor in real-time
sudo tail -f /var/log/fail2ban.log | grep -E "(Ban|Unban)"
```

## Best Practices

### 1. Regular Monitoring

- Check fail2ban status weekly
- Review logs for unusual activity
- Monitor your own IP to avoid accidental bans

### 2. Configuration Management

- Always test configuration changes in a safe environment
- Keep backups of working configurations
- Use the Ansible playbook for consistent deployments

### 3. Security Considerations

- Don't disable fail2ban on production servers
- Consider changing SSH port from default (22)
- Use key-based authentication instead of passwords
- Regularly update fail2ban and system packages

### 4. Documentation

- Keep a record of legitimate IPs that might trigger bans
- Document any custom rules or modifications
- Maintain emergency access procedures

### 5. Backup Plans

- Always have an alternative way to access your server
- Keep emergency contact information for your hosting provider
- Consider setting up a VPN for secure remote access

## Configuration Files Location

- **Main configuration**: `/etc/fail2ban/jail.local`
- **Default configuration**: `/etc/fail2ban/jail.conf` (don't modify this)
- **Logs**: `/var/log/fail2ban.log`
- **Service status**: `systemctl status fail2ban`

## Quick Reference Commands

```bash
# Most common commands
sudo fail2ban-client status sshd          # Check SSH jail status
sudo fail2ban-client set sshd unbanip IP  # Unban an IP
sudo fail2ban-client reload               # Reload configuration
sudo systemctl restart fail2ban           # Restart service
sudo tail -f /var/log/fail2ban.log       # Monitor logs

# Emergency commands
sudo systemctl stop fail2ban              # Stop fail2ban (removes all bans)
sudo systemctl start fail2ban             # Start fail2ban
sudo fail2ban-client -t                   # Test configuration
```

---

**Note**: Always ensure you have alternative access to your server before making significant changes to fail2ban configuration!
