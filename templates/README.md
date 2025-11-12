# ClickHouse Configuration Templates

This directory contains Jinja2 templates for ClickHouse configuration files.

## 🎯 Best Practice Approach

Following [ClickHouse official best practices](https://clickhouse.com/docs/operations/configuration-files), this role:

✅ **KEEPS default `config.xml` and `users.xml` UNCHANGED**  
✅ **Places customizations in `config.d/` and `users.d/` directories**  
✅ **Supports both XML and YAML formats**

### Why This Matters

> "To simplify updates and improve modularization, it is a best practice to keep the default `config.xml` file unmodified and place additional customization into `config.d/`."  
> — [ClickHouse Documentation](https://clickhouse.com/docs/operations/configuration-files)

**Benefits:**

- Package upgrades won't overwrite your configs
- Easier to manage and debug
- Modular configuration files
- Mix XML and YAML formats

---

## 📁 Template Structure

```
templates/
├── config.d/                       # Server configuration overrides
│   ├── network.xml.j2             # Network settings
│   ├── network.yaml.j2            # (YAML alternative)
│   ├── cluster.xml.j2             # Cluster configuration
│   ├── cluster.yaml.j2            # (YAML alternative)
│   ├── keeper.xml.j2              # Keeper client config
│   └── keeper.yaml.j2             # (YAML alternative)
│
├── keeper_config.xml.j2           # Keeper server config
├── keeper_config.yaml.j2          # (YAML alternative)
└── README.md                      # This file
```

---

## 🔧 Configuration Files

### 1. `config.d/network.{{ clickhouse_config_format }}.j2`

**Purpose:** Network binding configuration

**Deployed to:** `/etc/clickhouse-server/config.d/network.{{ clickhouse_config_format }}`

**Content:**

```xml
<clickhouse>
    <listen_host>{{ clickhouse_listen_host }}</listen_host>
</clickhouse>
```

**Variables:**

- `clickhouse_listen_host` - IP address to bind (auto-detected from inventory)

---

### 2. `config.d/cluster.{{ clickhouse_config_format }}.j2`

**Purpose:** Cluster topology and distributed DDL

**Deployed to:** `/etc/clickhouse-server/config.d/cluster.{{ clickhouse_config_format }}`

**Conditional:** Only deployed if `clickhouse_cluster_enabled` is `true`

**Features:**

- Auto-generates `remote_servers` configuration
- Creates macros for distributed DDL
- Configures distributed_ddl queue

**Variables:**

- Auto-detects shards and replicas from inventory

---

### 3. `config.d/keeper.{{ clickhouse_config_format }}.j2`

**Purpose:** ClickHouse Keeper client configuration (for Server nodes)

**Deployed to:** `/etc/clickhouse-server/config.d/keeper.{{ clickhouse_config_format }}`

**Conditional:** Only deployed if Keeper ensemble exists

**Content:**

```xml
<clickhouse>
    <zookeeper>
        <node>
            <host>keeper1_ip</host>
            <port>9181</port>
        </node>
        <!-- More keeper nodes -->
    </zookeeper>
</clickhouse>
```

---

### 4. `keeper_config.{{ clickhouse_config_format }}.j2`

**Purpose:** ClickHouse Keeper server configuration

**Deployed to:** `/etc/clickhouse-keeper/keeper_config.{{ clickhouse_config_format }}`

**Conditional:** Only deployed on Keeper nodes

**Features:**

- Raft consensus configuration
- Server ID and ensemble members
- Log and snapshot paths

**Variables:**

- `clickhouse_keeper_id` - Unique server ID from inventory
- `clickhouse_keeper_port` - Client port (default: 9181)
- `clickhouse_keeper_raft_port` - Raft port (default: 9234)

---

## ⚙️ Configuration Format

### Choose XML or YAML

Set the format in your playbook or inventory:

```yaml
clickhouse_config_format: xml   # Default
# OR
clickhouse_config_format: yaml
```

### XML Example

```xml
<?xml version="1.0"?>
<clickhouse>
    <listen_host>10.54.0.11</listen_host>
</clickhouse>
```

### YAML Example

```yaml
clickhouse:
  listen_host: 10.54.0.11
```

**Both formats are merged with default config.xml at runtime!**

---

## 🔄 How Configuration Merging Works

ClickHouse merges configurations in this order:

1. `/etc/clickhouse-server/config.xml` (default, unchanged)
2. Files in `/etc/clickhouse-server/config.d/` (alphabetical order)

**Example:**

```
config.xml                    # Default ClickHouse config (untouched)
config.d/
  ├── cluster.xml            # Our cluster config
  ├── keeper.xml             # Our keeper config
  └── network.xml            # Our network config
```

**Result:** All files are merged into effective configuration!

---

## 📊 Deployment Matrix

| Node Type       | config.d/network | config.d/cluster | config.d/keeper       | keeper_config |
| --------------- | ---------------- | ---------------- | --------------------- | ------------- |
| Server only     | ✅               | ✅ (if cluster)  | ✅ (if keeper exists) | ❌            |
| Keeper only     | ❌               | ❌               | ❌                    | ✅            |
| Server + Keeper | ✅               | ✅ (if cluster)  | ✅ (if keeper exists) | ✅            |

---

## 🎨 Customization Examples

### Add Custom Settings

**Option 1: Via Playbook**

```yaml
- hosts: clickhouse_cluster
  roles:
    - role: ansible-role-clickhouse
  tasks:
    - name: Add custom ClickHouse settings
      ansible.builtin.copy:
        content: |
          <clickhouse>
            <max_connections>8192</max_connections>
            <keep_alive_timeout>30</keep_alive_timeout>
          </clickhouse>
        dest: /etc/clickhouse-server/config.d/custom.xml
      notify: restart clickhouse-server
```

**Option 2: Via Template**

Create your own template in your playbook:

```
your-playbook/
  templates/
    custom_settings.xml.j2
```

Then deploy it:

```yaml
- name: Deploy custom settings
  ansible.builtin.template:
    src: custom_settings.xml.j2
    dest: /etc/clickhouse-server/config.d/custom.xml
  notify: restart clickhouse-server
```

---

## 🧪 Testing Configuration

### Verify Effective Configuration

```bash
# Check what ClickHouse will actually use
clickhouse-client --query "SELECT * FROM system.settings LIMIT 10"

# Check cluster configuration
clickhouse-client --query "SELECT * FROM system.clusters"

# Check merged configuration (preprocessed)
cat /var/lib/clickhouse/preprocessed_configs/config.xml
```

### Validate Before Applying

The role automatically validates configs via handler:

```yaml
- name: Validate ClickHouse Server config
  command: clickhouse-server --config-file=/etc/clickhouse-server/config.xml --test
```

---

## 📚 References

- [ClickHouse Configuration Files Documentation](https://clickhouse.com/docs/operations/configuration-files)
- [YAML Configuration Format](https://clickhouse.com/docs/operations/configuration-files#yaml-examples)
- [Configuration Merging Rules](https://clickhouse.com/docs/operations/configuration-files#merging-configuration)

---

## 🔐 Security Notes

**Default `config.xml` and `users.xml` are SAFE:**

- Provided by ClickHouse package
- Tested and maintained by ClickHouse team
- Automatically updated with package upgrades

**Our config.d/ files ONLY ADD/OVERRIDE specific settings:**

- Network binding
- Cluster topology
- Keeper ensemble
- Everything else uses ClickHouse defaults

---

## 🚀 Migration from Old Approach

If you were overwriting `config.xml`:

**❌ Old (Not Recommended):**

```yaml
- template:
    src: config.xml.j2
    dest: /etc/clickhouse-server/config.xml # Overwrites default!
```

**✅ New (Best Practice):**

```yaml
- template:
    src: config.d/network.xml.j2
    dest: /etc/clickhouse-server/config.d/network.xml # Adds to defaults!
```

**Benefits:**

- Package upgrades safe
- Easier to maintain
- Clear what you've customized
- Can mix XML and YAML

---

## 💡 Tips

1. **File naming:** Use descriptive names in `config.d/` (e.g., `network.xml`, `cluster.xml`)
2. **Alphabetical loading:** Files are loaded alphabetically, name accordingly
3. **Mix formats:** You can use both XML and YAML in config.d/
4. **Backup:** All templates use `backup: true` - old configs are saved as `.backup`
5. **Validation:** Configs are validated before restart via handlers

---

## 🎯 Summary

✅ Keep `/etc/clickhouse-server/config.xml` **UNCHANGED**  
✅ Keep `/etc/clickhouse-server/users.xml` **UNCHANGED**  
✅ Add customizations to `/etc/clickhouse-server/config.d/`  
✅ Add user customizations to `/etc/clickhouse-server/users.d/`  
✅ Choose XML or YAML format via `clickhouse_config_format`  
✅ Let ClickHouse merge everything automatically

**Result:** Clean, maintainable, upgrade-safe configuration! 🎉
