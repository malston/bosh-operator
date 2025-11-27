---
name: scale-deployment
description: Use when operators ask about scaling BOSH deployments, changing instance counts, or managing capacity. Provides guidance since scaling requires manifest changes.
---

# Scale BOSH Deployment

You are helping an operator understand and plan BOSH deployment scaling.

## Important Context

**BOSH scaling requires manifest changes and redeployment.** The bosh-mcp-server does not include deployment tools (`bosh deploy`), so this skill provides guidance and helps with temporary capacity management via stop/start operations.

## Scaling Guidance Workflow

### Step 1: Understand the Request

Clarify what the operator wants:

1. **Scale up** - Add more instances of a job
2. **Scale down** - Remove instances of a job
3. **Temporary reduction** - Stop instances without removing them
4. **Check current scale** - See instance counts

### Step 2: Show Current State

Get current instance counts:
```
bosh_instances(deployment: "<deployment>")
```

Present clearly:
```
Current instance counts for: cf

Job                 Instances   AZs
-----------------------------------------
diego_cell          3           z1, z2, z3
router              2           z1, z2
api                 1           z1
uaa                 1           z1
doppler             2           z1, z2
```

### Step 3: Explain Scaling Options

**For permanent scaling (up or down):**

```
To permanently change instance counts, you need to:

1. Edit the deployment manifest (or ops file)
2. Change the 'instances' value for the job
3. Run 'bosh deploy' with the updated manifest

Example ops file to scale diego_cells to 5:
---
- type: replace
  path: /instance_groups/name=diego_cell/instances
  value: 5

This operation is not available through the MCP server.
Do you have access to the deployment manifest?
```

**For temporary capacity reduction:**

```
To temporarily reduce capacity without manifest changes,
you can stop instances:

bosh_stop(deployment: "cf", job: "diego_cell/2")

This keeps the VM allocated but stops all processes.
To restore, use bosh_start.

Warning: Stopped instances still consume IaaS resources.
```

### Step 4: Temporary Scaling Operations

**To stop instances (reduce running capacity):**

```
bosh_stop(deployment: "<deployment>", job: "<job>/<index>")
```

Explain the impact:
```
Stopping diego_cell/2 will:
- Stop all processes on that VM
- Remove it from the pool of available cells
- Apps may be rescheduled to remaining cells

The VM remains allocated. To fully remove it, you need
to scale down via manifest change.
```

**To start stopped instances (restore capacity):**

```
bosh_start(deployment: "<deployment>", job: "<job>/<index>")
```

### Step 5: Capacity Planning Guidance

If the operator is planning scaling:

```
Capacity Planning Considerations:

For diego_cells:
- Each cell can run ~100-250 app instances (depends on memory)
- Recommend N+1 for high availability
- Scale based on actual memory pressure, not instance count

For routers:
- Scale based on requests per second
- Recommend at least 2 for HA
- Consider dedicated routes for high-traffic apps

For databases:
- Scaling often requires special procedures
- Consider read replicas vs primary scaling
- Consult your database release documentation
```

### Step 6: Pre-scaling Checks

Before scaling, recommend:

```
Before scaling, verify:

1. Check current resource usage:
   bosh_instances(deployment: "cf")
   Look at CPU and memory percentages

2. Check IaaS quotas:
   - Sufficient VM quota?
   - Sufficient IP addresses?
   - Sufficient disk quota?

3. Check cloud config:
   bosh_cloud_config()
   Verify AZ configuration and VM types
```

## Stop/Start Operations

### Stopping an Instance

```
bosh_stop(deployment: "<deployment>", job: "<job>")
```

Use cases:
- Temporary maintenance
- Cost savings (stop non-prod overnight)
- Controlled capacity reduction

### Starting an Instance

```
bosh_start(deployment: "<deployment>", job: "<job>")
```

Use cases:
- Restore after maintenance
- Bring capacity back online
- Recovery from planned stop

## Limitations

Be clear about what this skill cannot do:

```
Through the MCP server, I can:
- Show current instance counts
- Stop/start existing instances
- Help plan scaling changes

I cannot:
- Add new instances (requires manifest + deploy)
- Remove instances (requires manifest + deploy)
- Change VM types (requires manifest + deploy)
- Modify persistent disks (requires manifest + deploy)

For these operations, you'll need access to:
- The deployment manifest or ops files
- The BOSH CLI with deploy permissions
```

## Recommendations

Based on common patterns:

**Scaling diego_cells:**
```
Typical scaling triggers:
- Memory pressure > 70% sustained
- App staging queue growing
- Diego auction failures increasing

Safe scaling approach:
1. Add cells one AZ at a time
2. Wait for new cell to receive workload
3. Verify stability before adding more
```

**Scaling routers:**
```
Typical scaling triggers:
- CPU > 60% on routers
- Request latency increasing
- Connection errors rising

Recommendation: Scale in pairs to maintain HA
```
