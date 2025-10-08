# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.3] - 2025-10-08

### Added

- **Multi-cluster Support**: Configure multiple ClickHouse clusters on the same set of nodes
  - New inventory format using `clickhouse_clusters` dictionary
  - Automatic cluster topology generation from inventory
  - Support for nodes participating in multiple clusters
  - Macros generated for first cluster (alphabetical order)
- **User Management Configuration**: Auto-configure default user with necessary permissions
  - Enable `access_management` for user creation and management
  - Enable `named_collection_control` for named collections
  - Enable `show_named_collections` and `show_named_collections_secrets`
  - Allows `GRANT ALL ON *.* WITH GRANT OPTION` without errors
- **Example Inventories**: 
  - `hosts-multi-cluster.yml` - Full multi-cluster example
  - `example-add-second-cluster.yml.sample` - Adding second cluster to existing setup

### Fixed

- **XML Parse Error**: Fixed whitespace before `<?xml version="1.0"?>` in all templates
  - Added Jinja2 whitespace control (`{%-` and `-%}`) to strip whitespace
  - Fixed in `cluster.xml.j2`, `network.xml.j2`, `keeper.xml.j2`, `keeper_config.xml.j2`
- **Replication DNS Error**: Added `interserver_http_host` configuration in `network.xml.j2`
  - Fixes "Not found address of host" errors during replication
  - Uses `clickhouse_advertise_host` (auto-detected from inventory)
- **Replica Identification**: Changed macros to use numeric replica ID instead of hostname
  - Uses `clickhouse_replica` variable from inventory
  - More predictable and cleaner than hostnames

### Changed

- **Breaking: New Inventory Format**: 
  - Old format with `clickhouse_shard` and `clickhouse_replica` is removed
  - New format uses `clickhouse_clusters` dictionary:
    ```yaml
    clickhouse_clusters:
      cluster1:
        shard: 1
        replica: 1
    ```
- **Simplified Macros**: Only first cluster gets default macros (`{cluster}`, `{shard}`, `{replica}`)
  - Removed per-cluster macros with prefixes
  - Cleaner and more straightforward usage

### Documentation

- Updated README with multi-cluster examples and documentation
- Added user management section with SQL examples
- Updated all inventory examples to new format
- Improved SQL usage examples for multiple clusters

### Migration Guide

To migrate from v1.0.0 to v1.0.3:

**Old inventory format:**
```yaml
clickhouse_shard: 1
clickhouse_replica: 1
```

**New inventory format:**
```yaml
clickhouse_clusters:
  main:
    shard: 1
    replica: 1
```

## [1.0.0] - 2025-10-07

### Added

- Initial release of ansible-role-clickhouse
- Support for ClickHouse Server installation on Ubuntu 24.04 LTS
- Support for ClickHouse Keeper installation and clustering
- Flexible inventory structure (Kubespray-inspired)
- Auto-detection of cluster topology from inventory
- Support for standalone, replicated, and sharded deployments
- Configuration format flexibility (XML/YAML)
- Best practice configuration using `config.d/` and `users.d/` directories
- Complete reset playbook for cluster cleanup
- Comprehensive documentation and examples
- CI/CD pipeline with GitHub Actions
- IPv4-only support with automatic IPv6 handling

### Features

- **Deployment Modes**: Standalone, replicated, sharded clusters
- **ClickHouse Keeper**: Built-in coordination service support
- **Flexible Roles**: Nodes can run Server, Keeper, or both
- **Auto-detection**: Cluster topology from inventory variables
- **Config Formats**: Choose between XML or YAML configuration
- **Clean Reset**: Complete uninstall without OS reinstall
- **Production Ready**: Follows ClickHouse best practices

### Requirements

- Ansible 2.9+
- Ubuntu 24.04 LTS target hosts
- Python 3 on target hosts

[1.0.3]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.3
[1.0.0]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.0
