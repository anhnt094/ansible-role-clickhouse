# Quick Start Guide

Get ClickHouse up and running in minutes!

## Prerequisites

- Ansible 2.9+ installed on control machine
- Ubuntu 24.04 LTS target servers
- SSH access to target servers
- Python 3 on target servers

## Installation

```bash
ansible-galaxy install anhnt094.clickhouse
```

## Simple Standalone Setup

### 1. Create Inventory

```yaml
# inventory.yml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 192.168.1.10
  children:
    clickhouse_cluster:
      hosts:
        clickhouse1:
```

### 2. Create Playbook

```yaml
# deploy.yml
- hosts: clickhouse_cluster
  become: true
  roles:
    - anhnt094.clickhouse
```

### 3. Deploy

```bash
ansible-playbook -i inventory.yml deploy.yml
```

### 4. Verify

```bash
ssh user@192.168.1.10 "clickhouse-client --query 'SELECT version()'"
```

## Simple Cluster Setup (3 nodes)

### 1. Create Inventory

```yaml
# inventory.yml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 10.0.1.10
      access_ip: 10.0.1.10
      clickhouse_keeper_id: 1
      clickhouse_shard: 1
      clickhouse_replica: 1

    clickhouse2:
      ansible_host: 192.168.1.11
      ip: 10.0.1.11
      access_ip: 10.0.1.11
      clickhouse_keeper_id: 2
      clickhouse_shard: 1
      clickhouse_replica: 2

    clickhouse3:
      ansible_host: 192.168.1.12
      ip: 10.0.1.12
      access_ip: 10.0.1.12
      clickhouse_keeper_id: 3
      clickhouse_shard: 1
      clickhouse_replica: 3

  children:
    clickhouse_cluster:
      hosts:
        clickhouse1:
        clickhouse2:
        clickhouse3:

    clickhouse_keeper:
      hosts:
        clickhouse1:
        clickhouse2:
        clickhouse3:
```

### 2. Deploy

```bash
ansible-playbook -i inventory.yml deploy.yml
```

### 3. Verify Cluster

```bash
# Check cluster configuration
ssh user@192.168.1.10 "clickhouse-client --query 'SELECT * FROM system.clusters'"

# Check replication
ssh user@192.168.1.10 "clickhouse-client --query 'SELECT * FROM system.replicas'"

# Check Keeper
ssh user@192.168.1.10 "clickhouse-keeper-client -q 'ls /'"
```

## Common Operations

### Update Configuration

```bash
# Change to YAML format
ansible-playbook -i inventory.yml deploy.yml -e clickhouse_config_format=yaml

# Change cluster name
ansible-playbook -i inventory.yml deploy.yml -e clickhouse_cluster_name=production
```

### Reset Cluster

```bash
# With confirmation prompt
ansible-playbook -i inventory.yml reset.yml

# Skip confirmation (dangerous!)
ansible-playbook -i inventory.yml reset.yml -e reset_confirmation=yes
```

### Upgrade ClickHouse

```bash
# Update to specific version
ansible-playbook -i inventory.yml deploy.yml -e clickhouse_version=24.8.1.1
```

## Next Steps

- Read full documentation: [README.md](README.md)
- Explore examples: [test-playbooks/](test-playbooks/)
- Customize variables: [defaults/main.yml](defaults/main.yml)
- Join community: [GitHub Discussions](https://github.com/anhnt094/ansible-role-clickhouse/discussions)

## Troubleshooting

### SSH Connection Issues

```bash
# Disable host key checking temporarily
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.yml deploy.yml
```

### Permission Denied

```bash
# Use specific SSH key
ansible-playbook -i inventory.yml deploy.yml --private-key ~/.ssh/your-key.pem

# Use specific user
ansible-playbook -i inventory.yml deploy.yml -u ubuntu
```

### Check Syntax

```bash
ansible-playbook -i inventory.yml deploy.yml --syntax-check
```

### Dry Run

```bash
ansible-playbook -i inventory.yml deploy.yml --check
```

### Verbose Output

```bash
ansible-playbook -i inventory.yml deploy.yml -vvv
```

## Need Help?

- [Documentation](README.md)
- [GitHub Issues](https://github.com/anhnt094/ansible-role-clickhouse/issues)
- [Contributing Guide](CONTRIBUTING.md)
