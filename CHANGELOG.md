# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.5] - 2025-10-09

### Security

- **Network Access Control for Default User**
  - Added network restrictions for default ClickHouse user
  - New variable: `clickhouse_default_user_networks` (default: `["::1", "127.0.0.1"]`)
  - New variable: `clickhouse_default_user_networks_required` (default: `true`)
  - Default configuration restricts access to localhost only
  - Prevents unauthorized access from public networks
  - Validation task fails if `::0` (all networks) is detected
  - Can be disabled with `clickhouse_default_user_networks_required: false`

### Added

- **Network Restrictions Template**: Updated `templates/users.d/default.xml.j2`
  - Added `<networks>` section with configurable IP ranges
  - Supports both XML and YAML formats
  - Password remains empty (no password required from allowed networks)
- **Security Validation**: Pre-deployment check in `tasks/ubuntu-24.yml`
  - Validates network restrictions are configured
  - Provides clear error messages with configuration examples
  - Can be bypassed if needed for specific use cases

### Changed

- **Example Playbooks**: Updated with network security configuration
  - `test-playbooks/example-standalone.yml.sample`
  - `test-playbooks/example-cluster.yml.sample`

## [1.0.4] - 2025-10-08

### Added

- **Path Configuration Template**: New `config.d/path.xml.j2` template for centralized path management
  - Configures `path` (data directory): `/var/lib/clickhouse/`
  - Configures `tmp_path` (temporary data): `/var/lib/clickhouse/tmp/`
  - Configures `user_files_path` (user files): `/var/lib/clickhouse/user_files/`
  - Configures `format_schema_path` (format schemas): `/var/lib/clickhouse/format_schemas/`
  - Configures `top_level_domains_path` (TLD list): `/var/lib/clickhouse/top_level_domains/`
  - Configures `user_directories` (users.xml and local directory paths)
  - Support for both XML and YAML formats
  - Uses variables from `defaults/main.yml` for easy customization

### Changed

- **Deployment Tasks**: Updated `tasks/ubuntu-24.yml` to deploy path configuration
  - Added automatic deployment of `path.xml` or `path.yaml` to `config.d/`
  - Added cleanup task to remove old format files when switching between XML/YAML
  - Path config deployed before cluster config in deployment sequence

### Improved

- **Configuration Management**: Better organization of path-related settings
  - All path configurations in one dedicated template
  - Consistent with ClickHouse best practices (trailing slashes required)
  - Easy to override paths via inventory variables

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

[1.0.4]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.4
[1.0.3]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.3
[1.0.0]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.0
