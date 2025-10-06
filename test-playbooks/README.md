# Test Playbooks

This directory is for local testing only. All files here (except `.sample` files and this README) are ignored by git.

## Quick Reference

### Common Operations

```bash
# Install/Deploy ClickHouse
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-cluster.yml

# Reset ClickHouse (Complete Cleanup)
ansible-playbook -i test-playbooks/inventory.yml reset.yml
ansible-playbook -i test-playbooks/inventory.yml reset.yml -e reset_confirmation=yes  # Skip prompt

# Change Config Format
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml -e clickhouse_config_format=yaml

# Deploy Only Configuration
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml --tags config

# Dry Run (Check Mode)
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml --check
```

## How to Use

### 1. Create Inventory

Copy sample and update with your hosts:

```bash
cp test-playbooks/example-inventory.yml.sample test-playbooks/inventory.yml
# Edit inventory.yml with your actual IPs
```

Example inventory:

```yaml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 192.168.1.10
      access_ip: 192.168.1.10
      clickhouse_keeper_id: 1
      clickhouse_shard: 1
      clickhouse_replica: 1
  children:
    clickhouse_cluster:
      hosts:
        clickhouse1:
```

### 2. Create Playbook

Copy sample and customize:

```bash
cp test-playbooks/example-standalone.yml.sample test-playbooks/test-standalone.yml
# or
cp test-playbooks/example-cluster.yml.sample test-playbooks/test-cluster.yml
```

### 3. Run

```bash
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml
```

## Testing Workflow

### Full Test Cycle

```bash
# 1. Deploy ClickHouse cluster
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-cluster.yml

# 2. Verify installation
ssh user@clickhouse1 "clickhouse-client --query 'SELECT version()'"
ssh user@clickhouse1 "clickhouse-client --query 'SELECT * FROM system.clusters'"

# 3. Test data operations
ssh user@clickhouse1 "clickhouse-client --query 'CREATE DATABASE test'"
ssh user@clickhouse1 "clickhouse-client --query 'SHOW DATABASES'"

# 4. Reset cluster (clean uninstall)
ansible-playbook -i test-playbooks/inventory.yml reset.yml -e reset_confirmation=yes

# 5. Verify cleanup
ssh user@clickhouse1 "systemctl status clickhouse-server"  # Should fail
ssh user@clickhouse1 "ls /var/lib/clickhouse"  # Should not exist

# 6. Reinstall to verify clean state
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-cluster.yml
```

### Format Switching Test

```bash
# Deploy with XML
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml

# Switch to YAML
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml -e clickhouse_config_format=yaml

# Verify cleanup of old format
ssh user@clickhouse1 "ls -la /etc/clickhouse-server/config.d/"
# Should only see .yaml files, no .xml files

# Switch back to XML
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml -e clickhouse_config_format=xml
```

## Directory Structure

```
test-playbooks/
├── README.md                      # This file
├── .gitignore                     # Prevents committing sensitive data
├── example-inventory.yml.sample   # Example inventory configurations
├── example-standalone.yml.sample  # Standalone deployment example
├── example-cluster.yml.sample     # Cluster deployment example
├── test-reset.yml                 # Reset playbook wrapper
├── inventory.yml                  # Your actual inventory (git-ignored)
├── test-standalone.yml            # Your test playbook (git-ignored)
└── test-cluster.yml               # Your cluster test playbook (git-ignored)
```

## Tips

- **ansible.cfg**: The project root has `ansible.cfg` that disables SSH host key checking (like Kubespray)
- **No prompts**: You won't be asked "yes/no" when connecting to new hosts
- **Fast execution**: SSH pipelining is enabled for better performance
- **Verbose output**: Add `-v`, `-vv`, or `-vvv` for debugging
- **Dry run first**: Always test with `--check` flag before making changes
- **Tags**: Use `--tags config` or `--tags install` for partial runs
- **Reset safely**: The reset playbook requires confirmation unless you pass `-e reset_confirmation=yes`

## Notes

- All files in this directory are git-ignored except `.sample` files and README.md
- This prevents accidentally committing sensitive information (IPs, credentials)
- Always use `.sample` files as templates and create your own working copies
- The reset playbook completely removes ClickHouse - use it to start fresh without reinstalling the OS
