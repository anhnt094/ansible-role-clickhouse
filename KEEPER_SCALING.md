# ClickHouse Keeper Zero-Downtime Scaling Guide

This guide explains how to scale ClickHouse Keeper cluster without restarting existing nodes.

## Overview

The scaling process uses **dynamic reconfiguration** to add / remove Keeper nodes to a running cluster without downtime. This approach:

- ✅ Does not restart existing Keeper nodes
- ✅ Updates configuration files on all nodes for consistency
- ✅ Uses ClickHouse Keeper's built-in Raft reconfiguration API

## Prerequisites

- Existing Keeper cluster is healthy and has quorum
- New Keeper node(s) are provisioned and accessible
- Inventory file includes new nodes in `clickhouse_keeper` group
- Each Keeper node has unique `clickhouse_keeper_id` in inventory

## Scale Out (Adding Nodes)

### Example Scenario

You have 3 Keeper nodes (keeper1, keeper2, keeper3) and want to add keeper4.

### Step 0: Add New Node to Inventory

First, add the new keeper node to your Ansible inventory file:

```yaml
all:
  children:
    clickhouse_keeper:
      hosts:
        keeper1:
          ansible_host: 10.0.0.1
          ip: 10.0.0.1
          clickhouse_keeper_id: 1
        keeper2:
          ansible_host: 10.0.0.2
          ip: 10.0.0.2
          clickhouse_keeper_id: 2
        keeper3:
          ansible_host: 10.0.0.3
          ip: 10.0.0.3
          clickhouse_keeper_id: 3
        keeper4: # <-- NEW NODE
          ansible_host: 10.0.0.4
          ip: 10.0.0.4
          clickhouse_keeper_id: 4
```

**Important:**

- Each keeper node must have a unique `clickhouse_keeper_id`
- The `ip` or `access_ip` variable is used in keeper configuration
- Make sure the new node is accessible via SSH

### Step 1: Update Configuration Files

Run your **existing playbook** with the scale mode flag to deploy updated configuration to all nodes (old + new):

```bash
ansible-playbook -i inventory your-playbook.yml \
  -e clickhouse_keeper_scale_mode=true \
  --limit keeper1,keeper2,keeper3,keeper4
```

**What happens:**

- Configuration files are updated on all 4 nodes
- The new config includes all 4 nodes in `raft_configuration`
- **No services are restarted** (scale mode prevents this)
- Existing nodes continue running with old in-memory configuration

**Result:** All nodes have updated config files, but services are unchanged.

### Step 2: Start New Keeper Service

Start the Keeper service on the new node (keeper4):

```bash
ssh keeper4
systemctl daemon-reload
sudo systemctl start clickhouse-keeper
sudo systemctl enable clickhouse-keeper
```

### Step 3: Dynamic Reconfiguration

Add the new node to the cluster using any existing Keeper node:

```bash
echo "reconfig add server.4=10.0.0.4:9234" | \
  clickhouse-keeper-client
```

Replace:

- `server.4` with the server ID from inventory (`clickhouse_keeper_id`)
- `10.0.0.4` with the new node's IP address
- `9234` with the Raft port (default: 9234)

**What happens:**

- Node 4 joins the cluster automatically
- Cluster now has 4 active nodes

### Step 4: Restart New Node (Recommended)

Restart the new node to ensure clean Raft state:

```bash
ssh keeper4
systemctl restart clickhouse-keeper
```

**Why restart?**

- First start after `reconfig add` leaves the node in "catching up" state
- Restart ensures full Raft member capabilities (e.g., leadership transfer with `rqld`)
- The node will use already-synced data from disk, so restart is fast

**Verify:**

```bash
# Check cluster configuration
echo "get '/keeper/config'" | clickhouse-keeper-client
# Should show all 4 servers

# Check cluster status
echo "mntr" | clickhouse-keeper-client | grep followers
# Should show:
# zk_followers	3
# zk_synced_followers	3
```

### Step 5: Verify Cluster Health

Check that all nodes are healthy (run on all nodes):

```bash
echo "mntr" | clickhouse-keeper-client | grep -E "zk_server_state|zk_followers|zk_synced_followers"
```

Expected output:

- One node shows `zk_server_state=leader`
- Three nodes show `zk_server_state=follower`
- Leader shows `zk_followers=3` and `zk_synced_followers=3`

## Scale In (Removing Nodes)

This section explains how to safely remove Keeper nodes from a running cluster without downtime.

### Prerequisites for Scale In

**⚠️ CRITICAL: Maintain Quorum at All Times**

Before removing nodes, ensure you will maintain quorum after removal:

- **Minimum 3 nodes**: Never go below 3 nodes in production
- **Quorum calculation**: After removal, must have (N/2 + 1) nodes available
- **Examples**:
  - 5 nodes → can safely remove 2 (leaving 3)
  - 4 nodes → can safely remove 1 (leaving 3)
  - 3 nodes → **CANNOT remove any** (would break quorum)

### Scale In Workflow

**Example: Remove keeper4 from a 4-node cluster (4→3)**

### Step 1: Update Configuration Files

**⚠️ IMPORTANT: Update configs FIRST so ClickHouse Servers can reconnect to remaining Keepers!**

Remove the node from your inventory:

```yaml
all:
  children:
    clickhouse_keeper:
      hosts:
        keeper1:
          ansible_host: 10.0.0.1
          ip: 10.0.0.1
          clickhouse_keeper_id: 1
        keeper2:
          ansible_host: 10.0.0.2
          ip: 10.0.0.2
          clickhouse_keeper_id: 2
        keeper3:
          ansible_host: 10.0.0.3
          ip: 10.0.0.3
          clickhouse_keeper_id: 3
        # keeper4 REMOVED
```

Update config files on remaining nodes:

```bash
ansible-playbook -i inventory your-playbook.yml \
  -e clickhouse_keeper_scale_mode=true \
  --limit keeper1,keeper2,keeper3
```

**What happens:**

- Keeper configs updated (only 3 nodes in `raft_configuration`)
- ClickHouse Server configs updated (only 3 nodes in `keeper.xml`)
- **No services are restarted** (scale mode prevents this)
- ClickHouse Servers automatically reload config and reconnect to remaining Keepers

### Step 2: Dynamic Reconfiguration (Remove from Runtime)

Remove node 4 from the running cluster:

```bash
# Run on node 1, 2 or 3
echo "reconfig remove 4" | clickhouse-keeper-client
```

Replace `4` with the server ID you want to remove (`clickhouse_keeper_id`).

**What happens:**

- Leader removes node 4 from runtime cluster configuration
- Node 4 automatically steps down and stops participating
- Remaining nodes (1, 2, 3) continue operating
- Cluster now runs with 3 nodes

**Verify removal was successful:**

```bash
# Check cluster configuration
echo "get '/keeper/config'" | clickhouse-keeper-client

# Should show only 3 servers (node 4 is gone)
# server.1=10.0.0.1:9234;participant;1
# server.2=10.0.0.2:9234;participant;1
# server.3=10.0.0.3:9234;participant;1

# Check cluster status
echo "mntr" | clickhouse-keeper-client | grep zk_followers
# Should show: zk_followers=2
```

### Step 3: Stop the Removed Node

Now that the node has been removed from the cluster, you can safely stop its service:

```bash
ssh keeper4
sudo systemctl stop clickhouse-keeper
sudo systemctl disable clickhouse-keeper
```

**Optional: Clean up data (if decommissioning permanently):**

```bash
ssh keeper4
sudo rm -rf /var/lib/clickhouse/coordination/logs/*
sudo rm -rf /var/lib/clickhouse/coordination/snapshots/*
```

### Step 4: Verify Cluster Health

```bash
# Check all remaining nodes are healthy
for host in keeper1 keeper2 keeper3; do
  echo "=== $host ==="
  echo "mntr" | clickhouse-keeper-client --host $host --port 9181 | \
    grep -E "zk_server_state|zk_followers"
done
```

Expected output:

- One node: `zk_server_state=leader`
- Two nodes: `zk_server_state=follower`
- Leader: `zk_followers=2`

### Safety Best Practices

1. **Never remove the leader first**

   - Check who is the leader: `echo "mntr" | clickhouse-keeper-client | grep zk_server_state`
   - If you need to remove the leader, transfer leadership first:

     ```bash
     # SSH to a follower node you want to promote
     ssh keeper2

     # Transfer leadership to this node
     echo "rqld" | clickhouse-keeper-client
     ```

2. **Always maintain quorum**

   - 5 nodes → safe to have 3 (can lose 2)
   - 4 nodes → safe to have 3 (can lose 1)
   - 3 nodes → **DO NOT remove any**

3. **Wait between removals**

   - Check `zk_synced_followers` equals `zk_followers`
   - Ensure cluster is stable before next removal

4. **Verify after each step**
   - Use `reconfig` command to check cluster membership
   - Use `mntr` command to check cluster health

## Important Notes

### Quorum Requirements

- Keeper requires majority quorum: (N/2 + 1) nodes must be available
- 3 nodes: need 2 alive (can lose 1)
- 4 nodes: need 3 alive (can lose 1)
- 5 nodes: need 3 alive (can lose 2)
- 6 nodes: need 4 alive (can lose 2)

### Best Practices

**When to scale out:**

- Scale from 3 → 5 nodes (increases fault tolerance from 1 to 2 failures)
- Even numbers (4, 6) don't improve fault tolerance vs odd numbers (3, 5)
- Don't exceed 7 nodes (Raft performance degrades with large clusters)

**When to scale in:**

- Reduce costs when high availability requirements decrease
- Optimize for smaller clusters (5 → 3 nodes)
- Always maintain minimum 3 nodes for production

## Further Reading

- [ClickHouse Keeper Documentation](https://clickhouse.com/docs/en/guides/sre/keeper/clickhouse-keeper)
- [Dynamic Reconfiguration](https://clickhouse.com/docs/en/operations/clickhouse-keeper#reconfiguration)
- [Keeper Client](https://clickhouse.com/docs/en/operations/utilities/clickhouse-keeper-client)
