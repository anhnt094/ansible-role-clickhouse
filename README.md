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
# Multiple clusters configuration
# Define clusters in inventory host vars:
clickhouse_clusters:
  cluster1:
    shard: 1
    replica: 1
  cluster2:
    shard: 1
    replica: 2

# Auto-detected from inventory
clickhouse_cluster_enabled: auto
```

### ClickHouse Keeper

```yaml
clickhouse_keeper_enabled: auto # Auto-detected from inventory
clickhouse_keeper_port: 9181 # Client port
clickhouse_keeper_raft_port: 9234 # Raft consensus port
clickhouse_keeper_path: /var/lib/clickhouse/coordination
```

### User Management

The role automatically configures the default user with necessary permissions for user management:

```yaml
# Default user permissions (automatically configured)
# - access_management: Allows creating and managing users
# - named_collection_control: Allows managing named collections
# - show_named_collections: Allows viewing named collections
# - show_named_collections_secrets: Allows viewing collection secrets
```

This enables the default user to create additional users and grant permissions:

```sql
-- Create new admin user
CREATE USER admin IDENTIFIED BY 'secure_password';

-- Grant all permissions
GRANT ALL ON *.* TO admin WITH GRANT OPTION;
```

### Configuration Format

```yaml
clickhouse_config_format: "xml" # "xml" or "yaml"
```

### Server Configuration

```yaml
# Maximum number of simultaneously processed requests
clickhouse_max_concurrent_queries: 1000 # Default: 1000
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
- `clickhouse_clusters`: Dictionary of cluster configurations (see examples below)

### Host Groups

- `clickhouse_cluster`: Hosts running ClickHouse Server
- `clickhouse_keeper`: Hosts running ClickHouse Keeper

## Example Inventories

### 1. Standalone Server (Single Cluster)

```yaml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 192.168.1.10
      clickhouse_clusters:
        main:
          shard: 1
          replica: 1
  children:
    clickhouse_cluster:
      hosts:
        clickhouse1:
```

### 2. Replicated Cluster (1 shard, 3 replicas)

```yaml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 10.0.1.10
      access_ip: 10.0.1.10
      clickhouse_keeper_id: 1
      clickhouse_clusters:
        main:
          shard: 1
          replica: 1

    clickhouse2:
      ansible_host: 192.168.1.11
      ip: 10.0.1.11
      access_ip: 10.0.1.11
      clickhouse_keeper_id: 2
      clickhouse_clusters:
        main:
          shard: 1
          replica: 2

    clickhouse3:
      ansible_host: 192.168.1.12
      ip: 10.0.1.12
      access_ip: 10.0.1.12
      clickhouse_keeper_id: 3
      clickhouse_clusters:
        main:
          shard: 1
          replica: 3

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
      clickhouse_clusters:
        main:
          shard: 1
          replica: 1

    clickhouse2:
      ansible_host: 192.168.1.11
      ip: 10.0.1.11
      access_ip: 10.0.1.11
      clickhouse_keeper_id: 2
      clickhouse_clusters:
        main:
          shard: 1
          replica: 2

    # Shard 2
    clickhouse3:
      ansible_host: 192.168.1.12
      ip: 10.0.1.12
      access_ip: 10.0.1.12
      clickhouse_keeper_id: 3
      clickhouse_clusters:
        main:
          shard: 2
          replica: 1

    clickhouse4:
      ansible_host: 192.168.1.13
      ip: 10.0.1.13
      access_ip: 10.0.1.13
      clickhouse_clusters:
        main:
          shard: 2
          replica: 2

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

### 4. Multiple Clusters

Define multiple clusters on the same set of nodes:

```yaml
all:
  hosts:
    clickhouse1:
      ansible_host: 192.168.1.10
      ip: 10.0.1.10
      access_ip: 10.0.1.10
      clickhouse_keeper_id: 1
      clickhouse_clusters:
        dexmarket:
          shard: 1
          replica: 1
        analytics:
          shard: 1
          replica: 1

    clickhouse2:
      ansible_host: 192.168.1.11
      ip: 10.0.1.11
      access_ip: 10.0.1.11
      clickhouse_keeper_id: 2
      clickhouse_clusters:
        dexmarket:
          shard: 1
          replica: 2
        analytics:
          shard: 1
          replica: 2

    clickhouse3:
      ansible_host: 192.168.1.12
      ip: 10.0.1.12
      access_ip: 10.0.1.12
      clickhouse_keeper_id: 3
      clickhouse_clusters:
        dexmarket:
          shard: 2
          replica: 1
        analytics:
          shard: 1
          replica: 3

    clickhouse4:
      ansible_host: 192.168.1.13
      ip: 10.0.1.13
      access_ip: 10.0.1.13
      clickhouse_clusters:
        dexmarket:
          shard: 2
          replica: 2

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

This creates:

- **dexmarket** cluster: 2 shards, 2 replicas per shard (4 nodes)
- **analytics** cluster: 1 shard, 3 replicas (3 nodes, node4 excluded)

Generated macros on each node (uses first cluster in alphabetical order):

```xml
<macros>
  <cluster>analytics</cluster>
  <shard>1</shard>
  <replica>1</replica>
</macros>
```

**Note**: Macros use the first cluster name in alphabetical order. In this example, "analytics" comes before "dexmarket", so macros point to analytics cluster.

**Usage in SQL:**

```sql
-- Create table using macros (first cluster)
CREATE TABLE my_table ON CLUSTER analytics
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{database}/{table}/{shard}', '{replica}')
ORDER BY id;

-- For other clusters, use explicit values or database-only path
CREATE TABLE dexmarket_table ON CLUSTER dexmarket
ENGINE = ReplicatedMergeTree('/clickhouse/tables/dexmarket/{database}/{table}', '1')
ORDER BY id;

-- Or use database-based path (recommended for multi-cluster)
CREATE TABLE multi_table ON CLUSTER dexmarket
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{database}/{table}', '1')
ORDER BY id;
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
