# Ansible Role: ClickHouse

[![CI](https://github.com/anhnt094/ansible-role-clickhouse/workflows/CI/badge.svg)](https://github.com/anhnt094/ansible-role-clickhouse/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-anhnt094.clickhouse-blue.svg)](https://galaxy.ansible.com/anhnt094/clickhouse)

Deploys ClickHouse clusters with ClickHouse Keeper on Ubuntu 24.04 LTS. Supports standalone, replicated, and sharded configurations with flexible inventory structure.

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy install anhnt094.clickhouse
```

### From GitHub

```bash
ansible-galaxy install git+https://github.com/anhnt094/ansible-role-clickhouse.git
```

### Using `requirements.yml`

```yaml
# requirements.yml
roles:
  - name: anhnt094.clickhouse
    version: v1.0.0
```

Then install:

```bash
ansible-galaxy install -r requirements.yml
```

## Requirements

- Ansible 2.9+
- Ubuntu 24.04 LTS target hosts
- Python 3 on target hosts
- Network connectivity between cluster nodes

## Role Variables

### ClickHouse Version & Repository

```yaml
clickhouse_version: "latest" # ClickHouse version
clickhouse_repo_url: "deb https://packages.clickhouse.com/deb stable main"
```

### Network Configuration

```yaml
clickhouse_http_port: 8123 # HTTP API port
clickhouse_tcp_port: 9000 # Native protocol port
clickhouse_interserver_http_port: 9009 # Inter-replica communication
clickhouse_listen_host: "{{ ip }}" # Auto-detected from inventory
clickhouse_advertise_host: "{{ access_ip }}" # Auto-detected from inventory
```

### Cluster Configuration

```yaml
clickhouse_cluster_name: "cluster1" # Default cluster name
clickhouse_cluster_enabled: auto # Auto-detected from inventory
clickhouse_shard_id: "{{ clickhouse_shard }}" # From inventory host var
clickhouse_replica_id: "{{ clickhouse_replica }}"
```

### ClickHouse Keeper

```yaml
clickhouse_keeper_enabled: auto # Auto-detected from inventory
clickhouse_keeper_port: 9181 # Client port
clickhouse_keeper_raft_port: 9234 # Raft consensus port
clickhouse_keeper_path: /var/lib/clickhouse/coordination
```

### Configuration Format

```yaml
clickhouse_config_format: "xml" # "xml" or "yaml"
```

### Service Management

```yaml
clickhouse_service_enabled: true
clickhouse_service_state: started
```

## Inventory Structure

The role uses a Kubespray-inspired inventory structure:

### Host Variables

- `ansible_host`: SSH connection IP
- `ip`: Internal communication IP (bind address)
- `access_ip`: External/advertised IP (for cluster communication)
- `clickhouse_keeper_id`: Unique Keeper server ID (1, 2, 3...)
- `clickhouse_shard`: Shard number (1, 2, 3...)
- `clickhouse_replica`: Replica number within shard (1, 2, 3...)

### Host Groups

- `clickhouse_cluster`: Hosts running ClickHouse Server
- `clickhouse_keeper`: Hosts running ClickHouse Keeper

## Example Inventories

### 1. Standalone Server

```yaml
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

### 2. Replicated Cluster (3 replicas)

```yaml
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

### 3. Sharded Cluster (2 shards, 2 replicas each)

```yaml
all:
  hosts:
    # Shard 1
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

    # Shard 2
    clickhouse3:
      ansible_host: 192.168.1.12
      ip: 10.0.1.12
      access_ip: 10.0.1.12
      clickhouse_keeper_id: 3
      clickhouse_shard: 2
      clickhouse_replica: 1

    clickhouse4:
      ansible_host: 192.168.1.13
      ip: 10.0.1.13
      access_ip: 10.0.1.13
      clickhouse_shard: 2
      clickhouse_replica: 2

  children:
    clickhouse_cluster:
      hosts:
        clickhouse1:
        clickhouse2:
        clickhouse3:
        clickhouse4:

    clickhouse_keeper:
      hosts:
        clickhouse1:
        clickhouse2:
        clickhouse3:
```

## Example Playbooks

### Basic Installation

```yaml
- hosts: clickhouse_cluster
  become: true
  roles:
    - ansible-role-clickhouse
```

### With Custom Variables

```yaml
- hosts: clickhouse_cluster
  become: true
  vars:
    clickhouse_cluster_name: "production"
    clickhouse_config_format: "yaml"
  roles:
    - ansible-role-clickhouse
```

## Configuration Files

The role follows ClickHouse best practices by keeping default `config.xml` and `users.xml` unchanged and placing customizations in:

- `/etc/clickhouse-server/config.d/network.xml` - Network settings
- `/etc/clickhouse-server/config.d/cluster.xml` - Cluster topology
- `/etc/clickhouse-server/config.d/keeper.xml` - Keeper client config
- `/etc/clickhouse-keeper/keeper_config.xml` - Keeper server config

You can switch between XML and YAML formats by setting `clickhouse_config_format`.

## Cluster Features

- **Auto-detection**: Cluster topology detected from inventory
- **Flexible deployment**: Standalone, replicated, or sharded
- **Keeper integration**: Built-in ClickHouse Keeper for coordination
- **Mixed roles**: Nodes can run Server, Keeper, or both
- **Format flexibility**: Choose XML or YAML configuration format
- **IPv4 only**: Automatically handles IPv6-disabled environments

## Testing

See `test-playbooks/` directory for example inventories and playbooks:

```bash
# Test standalone deployment
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-standalone.yml

# Test cluster deployment
ansible-playbook -i test-playbooks/inventory.yml test-playbooks/test-cluster.yml
```

## Reset ClickHouse Cluster

To completely remove ClickHouse installation and data (similar to Kubespray reset):

```bash
# Interactive mode (will prompt for confirmation)
ansible-playbook -i inventory.yml reset.yml

# Non-interactive mode (skip confirmation - use with caution!)
ansible-playbook -i inventory.yml reset.yml -e reset_confirmation=yes
```

**What the reset playbook does:**

- Stops all ClickHouse services
- Removes all ClickHouse packages
- Deletes all data directories
- Deletes all log files
- Deletes all configuration files
- Removes ClickHouse user and group
- Cleans up APT cache and repositories

**⚠️ WARNING**: This will DELETE ALL DATA! Use with caution.

## License

MIT

## Author

Anh Nguyen
