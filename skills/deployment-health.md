---
name: deployment-health
description: Use when operators ask about deployment status, health, or want an overview of their BOSH infrastructure. Provides comprehensive health reports and recommendations.
---

# BOSH Deployment Health Check

You are helping an operator understand the health of their BOSH deployments.

## Prerequisites

Verify BOSH tools are available. If not, suggest running the `setup-bosh-mcp` skill.

## Health Check Workflow

### Step 1: Determine Scope

Ask if the operator wants:
1. **All deployments** - Overview of entire BOSH Director
2. **Specific deployment** - Deep dive into one deployment

If they ask generally ("how's my BOSH?", "deployment status"), show all deployments.

### Step 2: Gather Deployment Data

**List all deployments:**
```
bosh_deployments()
```

**For each deployment (or the specified one), get VM status:**
```
bosh_vms(deployment: "<deployment>")
```

**Get detailed instance info:**
```
bosh_instances(deployment: "<deployment>")
```

### Step 3: Check Recent Activity

**Look for recent errors:**
```
bosh_tasks(state: "error", limit: 5)
```

**Check for active operations:**
```
bosh_locks()
```

### Step 4: Analyze and Categorize

Categorize each deployment:

**Healthy:**
- All VMs running
- All processes up
- No recent errors

**Degraded:**
- Some instances unhealthy but deployment functional
- Non-critical processes down
- Recent task errors but recovered

**Critical:**
- Core components down
- Multiple VMs failing
- Deployment not functional

### Step 5: Present Summary

Format the report clearly:

```
BOSH Infrastructure Health Report
=================================

Overall Status: DEGRADED

Deployments: 3
- cf: DEGRADED
- redis: HEALTHY
- mysql: HEALTHY

---

Deployment: cf
Status: DEGRADED
VMs: 12/13 running

Unhealthy Instances:
- diego_cell/2: process_state=failing
  - rep: stopped
  - garden: running
  - route_emitter: running

Recent Errors: 1 task failed in last 24h
- Task 456: "run errand smoke_tests" (4 hours ago)

Recommendation: Run troubleshoot-vm for diego_cell/2

---

Deployment: redis
Status: HEALTHY
VMs: 2/2 running
All processes healthy

---

Deployment: mysql
Status: HEALTHY
VMs: 3/3 running
All processes healthy
```

### Step 6: Infrastructure Overview

If requested or relevant, include:

**Stemcells:**
```
bosh_stemcells()
```

Report:
```
Stemcells:
- ubuntu-jammy/1.200 (used by: cf, redis, mysql)
- ubuntu-jammy/1.199 (unused - consider cleanup)
- ubuntu-bionic/1.150 (unused - outdated)

Recommendation: Clean up unused stemcells to save disk space
```

**Releases:**
```
bosh_releases()
```

Report which releases are in use and which could be cleaned up.

### Step 7: Offer Next Steps

Based on findings, offer relevant actions:

**If unhealthy VMs found:**
```
Would you like me to troubleshoot the unhealthy VMs?
```

**If recent errors:**
```
Would you like to see details about the failed tasks?
```

**If all healthy:**
```
Everything looks good! Is there anything specific you'd like to check?
```

## Quick Health Check

For a fast status check, use this abbreviated flow:

1. `bosh_deployments()` - Get deployment list
2. `bosh_vms(deployment: "X")` for each - Quick VM status
3. Report only issues

```
Quick Health Check
==================
cf: 12/13 VMs healthy (1 issue)
redis: 2/2 VMs healthy
mysql: 3/3 VMs healthy

1 issue found. Run full health check for details?
```

## Drill-Down Options

Offer to go deeper:

- **Process details**: Show CPU/memory usage from `bosh_instances`
- **Task history**: Show recent operations via `check-tasks` skill
- **Configuration**: Show cloud/runtime config via `bosh_cloud_config`, `bosh_runtime_config`
- **Variables**: List deployment variables via `bosh_variables`

## Monitoring Recommendations

If the operator asks about ongoing monitoring:

```
For continuous monitoring, consider:
1. BOSH Health Monitor alerts
2. Prometheus + BOSH exporter
3. Datadog/New Relic integrations
4. Periodic health checks via this skill

Would you like information about any of these options?
```
