---
name: restart-deployment
description: Use when operators need to restart BOSH deployments, jobs, or instances. Guides through safe rolling restarts with pre-flight checks and post-restart verification.
---

# Restart BOSH Deployment

You are helping an operator safely restart BOSH deployments, jobs, or instances.

## Prerequisites

Verify BOSH tools are available. If not, suggest running the `setup-bosh-mcp` skill.

## Restart Workflow

### Step 1: Clarify Scope

Ask the operator what they want to restart:

1. **Entire deployment** - All jobs and instances
2. **Specific job** - All instances of a job (e.g., all routers)
3. **Single instance** - One specific VM (e.g., router/0)

If they say "restart cf" or similar, clarify:
```
Do you want to restart:
1. The entire cf deployment (all VMs)
2. A specific job type (e.g., all diego_cells)
3. A single instance (e.g., diego_cell/0)
```

### Step 2: Pre-flight Checks

**Check deployment health:**
```
bosh_vms(deployment: "<deployment>")
```

Warn if any VMs are already unhealthy:
```
Warning: 2 VMs are currently unhealthy in this deployment.
Restarting may not fix the underlying issue.
Consider running troubleshoot-vm first.
```

**Check for active locks:**
```
bosh_locks()
```

If locks exist:
```
Warning: Deployment is currently locked by task <id>.
Wait for the task to complete before restarting.
```

**Check instance count:**
```
bosh_instances(deployment: "<deployment>")
```

Report the impact:
```
This will restart 8 VMs across 3 availability zones.
Estimated duration: 15-20 minutes.
```

### Step 3: Assess Risk

**For production deployments, warn about:**

- **Diego cells**: "Restarting diego_cells will cause running apps to be rescheduled. There may be brief downtime for apps with single instances."

- **Routers**: "Restarting routers will briefly interrupt HTTP traffic. Ensure you have multiple router instances."

- **Database jobs**: "Restarting database VMs may cause brief connection errors. Ensure clients can reconnect."

- **Singleton jobs**: "This job has only 1 instance. Restart will cause downtime."

### Step 4: Recommend Best Practices

For large deployments, recommend:

```
For a safer restart, I recommend:
1. Restart one AZ at a time
2. Verify health between AZs
3. Consider doing this during a maintenance window

Do you want to proceed with a full restart, or restart AZ-by-AZ?
```

### Step 5: Confirm and Execute

Present the final action:

```
Ready to restart:
- Deployment: cf
- Scope: All router instances (3 VMs)
- Impact: Brief HTTP traffic interruption

This operation requires confirmation. Proceed?
```

**For restart:**
```
bosh_restart(deployment: "<deployment>", job: "<job>")
```

**For single instance:**
```
bosh_restart(deployment: "<deployment>", job: "<job>/<index>")
```

The operation will return a confirmation token. After the operator confirms, the restart will execute.

### Step 6: Monitor Progress

The `bosh_restart` tool waits for task completion by default. While waiting:

```
Restart in progress...
Task ID: 1234
Status: processing

The restart is executing. This typically takes 2-5 minutes per instance.
```

### Step 7: Verify Success

After completion, verify the deployment is healthy:

```
bosh_instances(deployment: "<deployment>")
```

Report results:
```
Restart completed successfully!

All 3 router instances are now running:
- router/0: running (uptime: 30s)
- router/1: running (uptime: 45s)
- router/2: running (uptime: 60s)
```

If any instances failed to restart:
```
Warning: router/1 did not restart cleanly.
Process state: failing

Would you like me to troubleshoot this instance?
```

## Safety Guardrails

**ALWAYS:**
- Check locks before restarting
- Warn about impact to running workloads
- Verify health before and after
- Recommend AZ-by-AZ for large deployments

**NEVER:**
- Restart without confirming scope
- Ignore unhealthy VMs in pre-flight
- Proceed if deployment is locked
- Restart stateful jobs without explicit warning

## Rollback

If restart causes issues:
- Individual instance problems: Use `troubleshoot-vm` skill
- Widespread failures: May need to investigate via `bosh ssh` or check IaaS
- Configuration issues: May need to redeploy from last known good state
