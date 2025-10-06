# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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

[1.0.0]: https://github.com/anhnt094/ansible-role-clickhouse/releases/tag/v1.0.0
