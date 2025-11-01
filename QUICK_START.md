# Quick Start Guide

This guide will help you get started with the VPS Update playbook in just a few minutes.

## ⚠️ Supported Systems

This playbook works **exclusively with Debian/Ubuntu** systems:
- ✅ Debian 11 (Bullseye), 12 (Bookworm), 13 (Trixie)
- ✅ Ubuntu 20.04 LTS, 22.04 LTS, 24.04 LTS
- ❌ NOT supported: RedHat, CentOS, Rocky, or other distributions

## Prerequisites Check

Before you begin, ensure you have:

```bash
# Check Ansible version (should be >= 2.10)
ansible --version

# Check Python version (should be >= 3.8)
python3 --version
```

## Installation Steps

### 1. Install Python Dependencies (Optional)

For development and testing:

```bash
pip install -r requirements.txt
```

### 2. Install Ansible Collections

```bash
ansible-galaxy collection install -r requirements.yml
```

### 3. Setup Inventory

Copy and customize the inventory file:

```bash
cp inventory.ini.example inventory.ini
nano inventory.ini  # or use your preferred editor
```

Add your servers (Debian/Ubuntu only):

```ini
[debian]
server1.example.com ansible_user=your_username
server2.example.com ansible_user=your_username

[ubuntu]
server3.example.com ansible_user=your_username

[all:vars]
ansible_python_interpreter=/usr/bin/python3
# Uncomment if using SSH keys:
# ansible_ssh_private_key_file=~/.ssh/your_key
```

### 4. Test Connectivity

```bash
ansible all -i inventory.ini -m ping
```

Expected output:
```
server1.example.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Basic Usage

### Update All Servers (with automatic reboot if needed)

```bash
ansible-playbook playbooks/update.yml -i inventory.ini
```

### Update Without Rebooting

```bash
ansible-playbook playbooks/update.yml -i inventory.ini \
  --extra-vars "system_update_auto_reboot=false"
```

### Update Specific Group

```bash
# Only Debian servers
ansible-playbook playbooks/update.yml -i inventory.ini --limit debian

# Only Ubuntu servers
ansible-playbook playbooks/update.yml -i inventory.ini --limit ubuntu

# Specific host
ansible-playbook playbooks/update.yml -i inventory.ini --limit server1.example.com
```

### Dry Run (Check Mode)

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --check
```

## Advanced Usage

### Hold Specific Packages

Create `group_vars/all/updates.yml`:

```yaml
system_update_packages_hold:
  - docker-ce
  - postgresql-15
  - nginx
```

Then run normally:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini
```

### Using Tags

```bash
# Only update packages, don't check for reboot
ansible-playbook playbooks/update.yml --tags packages

# Full update including reboot checks
ansible-playbook playbooks/update.yml --tags update

# Only run reboot checks (requires previous update)
ansible-playbook playbooks/update.yml --tags reboot
```

### Using Sudo Password

If your user requires a password for sudo:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --ask-become-pass
```

### Verbose Output

For debugging:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini -v
# or -vv, -vvv for more verbosity
```

## Scheduling with Cron

To run updates automatically:

```bash
# Edit crontab
crontab -e

# Add this line to run every Sunday at 3 AM
0 3 * * 0 cd /path/to/vps-update && ansible-playbook playbooks/update.yml -i inventory.ini >> /var/log/ansible-updates.log 2>&1
```

## Development and Testing

### Run Linting

```bash
# YAML linting
yamllint .

# Ansible linting
ansible-lint
```

### Run Molecule Tests

```bash
cd roles/system_update
molecule test
```

### Test Single Platform

```bash
cd roles/system_update
molecule test --scenario-name default -- -e molecule_platform=debian-bullseye
```

## Troubleshooting

### SSH Connection Issues

```bash
# Test SSH manually
ssh -i ~/.ssh/your_key user@server.example.com

# Check Ansible can connect
ansible all -i inventory.ini -m ping -vvv
```

### Permission Denied

```bash
# Make sure your user has sudo rights on the target server
# Test manually:
ssh user@server.example.com 'sudo whoami'
```

### Package Manager Locked

If you get errors about package manager locks, the playbook will automatically wait up to 5 minutes for the lock to be released. If you need to check manually:

```bash
# On target server (Debian/Ubuntu):
sudo lsof /var/lib/dpkg/lock-frontend
sudo lsof /var/lib/apt/lists/lock
sudo lsof /var/cache/apt/archives/lock

# Kill the locking process if safe (e.g., unattended-upgrades)
sudo killall apt-get
sudo killall unattended-upgrade
```

## Advanced Features

### Batch Control

Update servers in smaller batches (more conservative):
```bash
ansible-playbook playbooks/update.yml -i inventory.ini \
  --extra-vars "update_serial_batch=1"  # One at a time
```

### Enable Reporting

```bash
ansible-playbook playbooks/update.yml -i inventory.ini \
  --extra-vars "system_update_report_enabled=true"
```

Reports are saved to: `/var/log/ansible-update-<date>.json` on each server.

### Enable Notifications

Set up webhook in `group_vars/all/updates.yml`:
```yaml
system_update_notify_enabled: true
system_update_notify_webhook: "https://your-webhook-url"
```

## Next Steps

- Read the full [README.md](README.md) for detailed documentation
- Check [roles/system_update/README.md](roles/system_update/README.md) for all role variables
- Review [CHANGELOG.md](CHANGELOG.md) for version history
- Set up pre-commit hooks: `pip install pre-commit && pre-commit install`

## Getting Help

If you encounter issues:

1. Check the [Troubleshooting](README.md#troubleshooting) section in README.md
2. Run with verbose output: `-vvv`
3. Check target server logs: `/var/log/syslog` or `/var/log/messages`
4. Review Ansible documentation: https://docs.ansible.com/

## Security Best Practices

1. **Never commit** `inventory.ini` with real server information
2. Use Ansible Vault for sensitive data:
   ```bash
   ansible-vault encrypt inventory.ini
   ansible-playbook playbooks/update.yml --ask-vault-pass
   ```
3. Use SSH keys instead of passwords
4. Limit playbook execution to specific users/groups
5. Regularly review and update the playbook

---

Happy automating! 🚀

