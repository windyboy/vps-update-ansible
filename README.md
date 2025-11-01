# VPS Update Playbook

An Ansible playbook to automatically update packages and optionally reboot Debian/Ubuntu VPS servers with intelligent batch control and safety checks.

## ⚠️ Important Notice

**This playbook is optimized exclusively for Debian/Ubuntu systems.**

### Supported Distributions
- ✅ Debian 11 (Bullseye)
- ✅ Debian 12 (Bookworm)
- ✅ Debian 13 (Trixie)
- ✅ Ubuntu 20.04 LTS (Focal)
- ✅ Ubuntu 22.04 LTS (Jammy)
- ✅ Ubuntu 24.04 LTS (Noble)

### Not Supported
- ❌ RedHat, CentOS, Rocky Linux, Alma Linux
- ❌ openSUSE, Arch, Alpine, or other distributions

The playbook will automatically verify that all target hosts are Debian-based and fail gracefully if not.

## Features

- ✅ **Batch control** - Update servers in groups (default 25% at a time) to avoid full outage
- ✅ **Intelligent reboot detection** - Uses native `/var/run/reboot-required` for accurate decisions
- ✅ **Pre-flight checks** - Verifies disk space, checks for locked package managers
- ✅ **APT optimization** - Uses `dist-upgrade` with conflict resolution
- ✅ **Configurable behavior** - Control reboot, batch size, and update strategy
- ✅ **Tagged tasks** - Selective execution with granular control
- ✅ **Reporting & notifications** - Optional JSON reports and webhook alerts
- ✅ **Idempotent operations** - Safe to run repeatedly

## Prerequisites

- Ansible >= 2.10
- Python 3 on target hosts
- SSH access to target servers
- Root or sudo privileges on target servers

## Quick Start

### 1. Setup Inventory

Copy the example inventory file and customize it:

```bash
cp inventory.ini.example inventory.ini
# Edit inventory.ini with your server details
```

Example inventory structure:

```ini
[debian]
server1.example.com ansible_user=admin
server2.example.com ansible_user=admin

[ubuntu]
server3.example.com ansible_user=admin

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 2. Configure Variables (Optional)

You can override default variables in your playbook or inventory:

```yaml
system_update_auto_reboot: true          # Auto reboot if needed
system_update_reboot_timeout: 600        # Reboot timeout in seconds
update_serial_batch: "25%"               # Batch size (servers per round)
system_update_pre_check_enabled: true    # Run pre-flight checks
system_update_packages_hold: []          # Packages to hold (not update)
```

### 3. Run the Playbook

#### Update all servers with auto-reboot:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini
```

#### Update without rebooting:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --extra-vars "system_update_auto_reboot=false"
```

#### Update specific hosts:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --limit debian
```

#### Using tags:

```bash
# Only update packages, skip reboot checks
ansible-playbook playbooks/update.yml --tags packages

# Full update with reboot
ansible-playbook playbooks/update.yml --tags update
```

## Configuration

### Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `system_update_auto_reboot` | `true` | Automatically reboot if required |
| `system_update_reboot_timeout` | `600` | Reboot timeout in seconds |
| `system_update_post_reboot_delay` | `30` | Wait time after reboot (seconds) |
| `update_serial_batch` | `"25%"` | Batch size for updates |
| `system_update_pre_check_enabled` | `true` | Enable pre-flight checks |
| `system_update_min_free_space_mb` | `1024` | Minimum free space required (MB) |
| `system_update_packages_hold` | `[]` | List of packages to hold (not update) |
| `system_update_report_enabled` | `false` | Enable JSON reporting |
| `system_update_notify_enabled` | `false` | Enable webhook notifications |

See `roles/system_update/defaults/main.yml` for all available variables and detailed documentation.

### Example: Control Batch Size

Update servers conservatively (one at a time):
```bash
ansible-playbook playbooks/update.yml -i inventory.ini \
  --extra-vars "update_serial_batch=1"
```

Update more aggressively (half at a time):
```bash
ansible-playbook playbooks/update.yml -i inventory.ini \
  --extra-vars "update_serial_batch='50%'"
```

### Example: Hold Specific Packages

Create a file `group_vars/all/updates.yml`:

```yaml
system_update_packages_hold:
  - docker-ce
  - postgresql-15
  - nginx
```

## Project Structure

```
vps-update/
├── ansible.cfg              # Ansible configuration
├── inventory.ini.example    # Example inventory template
├── playbooks/
│   └── update.yml          # Main playbook
├── roles/
│   └── system_update/      # Update role
│       ├── defaults/       # Default variables (detailed docs)
│       ├── meta/           # Role metadata
│       ├── molecule/       # Testing scenarios
│       ├── tasks/          # Main tasks
│       │   ├── main.yml    # Main update workflow
│       │   ├── precheck.yml # Pre-flight checks
│       │   └── report.yml  # Reporting & notifications
│       ├── vars/           # OS-specific variables
│       └── README.md       # Role documentation
├── .pre-commit-config.yaml # Code quality automation
└── CHANGELOG.md            # Version history
```

## Debian/Ubuntu Specific Features

### 1. Native Reboot Detection
Uses Debian/Ubuntu's native `/var/run/reboot-required` file for accurate reboot detection. The playbook will also show which packages require a reboot.

### 2. APT Configuration Management
- Automatic handling of configuration file conflicts (`force-confdef,force-confold`)
- Cache optimization with configurable timeout
- Support for `dist-upgrade` with full dependency resolution

### 3. Package Hold Support
Prevent specific packages from being updated:
```yaml
# group_vars/all.yml
system_update_packages_hold:
  - docker-ce
  - postgresql-*
  - kernel*
```

### 4. Batch Updates with Safety
Control how many servers update at once to prevent service disruption:
- `serial: "25%"` - Update 25% of servers at a time (default)
- `serial: 1` - One server at a time (safest)
- `serial: "50%"` - Half of servers at a time (faster)

### 5. Pre-Flight Checks
- Disk space verification (/ and /var)
- APT/dpkg lock detection and waiting
- Held package detection
- APT sources validity check

## Troubleshooting

### Permission Denied

Make sure your user has sudo privileges:

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --ask-become-pass
```

### SSH Key Issues

```bash
# Test connectivity
ansible all -i inventory.ini -m ping

# Use specific SSH key
ansible-playbook playbooks/update.yml -i inventory.ini --private-key ~/.ssh/your_key
```

### Check Mode (Dry Run)

```bash
ansible-playbook playbooks/update.yml -i inventory.ini --check
```

## License

MIT

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Author

Created by windy
