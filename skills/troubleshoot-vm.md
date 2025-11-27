---
name: troubleshoot-vm
description: Use when VMs are unhealthy, processes are failing, instances are unresponsive, or operators ask about diagnosing BOSH deployment issues. Guides through diagnosis and remediation.
---

# Troubleshoot BOSH VM

You are helping an operator diagnose and fix unhealthy VMs in a BOSH deployment.

## Prerequisites

First, verify BOSH tools are available. If `bosh_vms` or `bosh_instances` tools are not available, suggest running the `setup-bosh-mcp` skill.

## Troubleshooting Workflow

### Step 1: Identify the Target

If the operator hasn't specified a deployment or VM:

1. Use `bosh_deployments` to list all deployments
2. Ask which deployment they want to troubleshoot
3. If they mention a specific job/instance (e.g., "diego_cell/2"), note it for later

### Step 2: Assess VM Health

Use `bosh_vms` with the deployment name to get VM status:

```
bosh_vms(deployment: "<deployment-name>")
```

Look for VMs where:
- `process_state` is not "running"
- `state` is not "started"

Report findings clearly:
```
Deployment: cf
Total VMs: 13
Healthy: 12
Unhealthy: 1

Problem VMs:
- diego_cell/2 (10.0.1.12): process_state=failing
```

### Step 3: Get Detailed Process Information

For unhealthy VMs, use `bosh_instances` to see process-level details:

```
bosh_instances(deployment: "<deployment-name>")
```

Identify which specific processes are failing:
```
diego_cell/2:
  - rep: stopped (was running for 3 days)
  - garden: running
  - route_emitter: running
```

### Step 4: Check Recent Task History

Use `bosh_tasks` to see if recent operations caused the issue:

```
bosh_tasks(deployment: "<deployment-name>", state: "error", limit: 5)
```

Look for:
- Failed deployments
- Failed errands
- Interrupted operations

### Step 5: Diagnose the Problem

Based on findings, categorize the issue:

**Single process down, VM healthy:**
- Likely a process crash
- Recommend: `bosh_restart` for the specific job

**Entire VM unresponsive:**
- Could be IaaS issue, agent crash, or resource exhaustion
- Recommend: `bosh_recreate` for the specific instance

**Multiple VMs in same AZ failing:**
- Likely an IaaS or network issue
- DO NOT auto-remediate
- Recommend: Check IaaS console, network connectivity

**All instances of a job failing:**
- Likely a configuration or release issue
- Recommend: Review recent changes, check logs via `bosh ssh`

### Step 6: Recommend Action

Present the recommended action with context:

```
Diagnosis: The rep process on diego_cell/2 has crashed.

Recommended action: Restart the diego_cell job on instance 2
This will restart all processes on that instance without recreating the VM.

Do you want me to proceed?
```

### Step 7: Execute Remediation

**For restart:**
```
bosh_restart(deployment: "<deployment>", job: "<job>/<index>")
```

**For recreate:**
```
bosh_recreate(deployment: "<deployment>", job: "<job>", index: "<index>")
```

Note: These operations will trigger a confirmation token. The operator must confirm before execution.

### Step 8: Verify Fix

After the task completes:

1. Use `bosh_instances` to verify the VM is now healthy
2. Check that all processes are running
3. If still unhealthy, escalate to deeper investigation

## Safety Guardrails

**ALWAYS:**
- Check `bosh_locks` before attempting operations
- Warn about bootstrap instances before recreating
- Get explicit confirmation before recreating multiple VMs

**NEVER:**
- Recreate multiple VMs without explicit approval
- Assume recreation is safe for stateful jobs (databases, etc.)
- Ignore patterns (multiple failures in same AZ = IaaS issue)

## Escalation

If troubleshooting doesn't resolve the issue:
- Suggest checking VM logs via `bosh ssh`
- Recommend reviewing the BOSH Director logs
- Suggest the `check-tasks` skill for detailed task output
