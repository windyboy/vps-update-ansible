# Ansible Role: system_update

An Ansible role to update packages and optionally reboot Debian/Ubuntu VPS servers with intelligent batch control and comprehensive pre-checks.

## ⚠️ Debian/Ubuntu Only

This role is **exclusively designed for Debian and Ubuntu systems**. It uses APT-specific features and Debian's native reboot detection mechanisms.

## Requirements

- Ansible >= 2.10
- Python 3 on target hosts
- Root or sudo access on target hosts

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# APT configuration
system_update_cache_valid_time: 3600
system_update_force_apt_get: true

# Update strategy
system_update_auto_reboot: true
system_update_reboot_timeout: 600
system_update_post_reboot_delay: 30

# Batch control
update_serial_batch: "25%"
update_max_fail_percentage: 10

# Pre-checks
system_update_pre_check_enabled: true
system_update_min_free_space_mb: 1024

# Reporting & Notification
system_update_report_enabled: false
system_update_report_file: "/var/log/ansible-update-{{ ansible_date_time.date }}.json"
system_update_notify_enabled: false
system_update_notify_webhook: ""

# Optional: Package hold
system_update_packages_hold: []
```

## Dependencies

None.

## Example Playbook

### Basic usage (update all servers):

```yaml
---
- name: Update all servers
  hosts: all
  become: true
  gather_facts: true
  
  roles:
    - role: system_update
```

### Update without automatic reboot:

```yaml
---
- name: Update servers without reboot
  hosts: all
  become: true
  gather_facts: true
  
  roles:
    - role: system_update
      vars:
        system_update_auto_reboot: false
```

### Hold specific packages:

```yaml
---
- name: Update with package holds
  hosts: all
  become: true
  gather_facts: true
  
  roles:
    - role: system_update
      vars:
        system_update_packages_hold:
          - docker-ce
          - postgresql-*
          - nginx
```

### Enable reporting and notifications:

```yaml
---
- name: Update with reporting
  hosts: all
  become: true
  gather_facts: true
  
  roles:
    - role: system_update
      vars:
        system_update_report_enabled: true
        system_update_notify_enabled: true
        system_update_notify_webhook: "https://your-webhook-url"
```

### Using tags:

```bash
# Only update packages, skip reboot
ansible-playbook playbooks/update.yml --tags update,packages --skip-tags reboot

# Only reboot if needed
ansible-playbook playbooks/update.yml --tags reboot

# Full update with reboot
ansible-playbook playbooks/update.yml --tags update
```

## Supported Platforms

- ✅ Debian 11 (Bullseye)
- ✅ Debian 12 (Bookworm)
- ✅ Debian 13 (Trixie)
- ✅ Ubuntu 20.04 LTS (Focal)
- ✅ Ubuntu 22.04 LTS (Jammy)
- ✅ Ubuntu 24.04 LTS (Noble)

**Not supported**: RedHat, CentOS, Rocky Linux, or any non-Debian-based distributions.

## License

MIT

## Author Information

This role was created by windy.

