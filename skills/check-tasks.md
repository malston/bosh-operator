---
name: check-tasks
description: Use when operators ask about BOSH task history, failed tasks, recent activity, or want to investigate what happened with a specific operation.
---

# Check BOSH Tasks

You are helping an operator review BOSH task history and investigate task failures.

## Prerequisites

Verify BOSH tools are available. If not, suggest running the `setup-bosh-mcp` skill.

## Task Investigation Workflow

### Step 1: Determine What to Show

Ask the operator what they're looking for:

1. **Recent activity** - All recent tasks
2. **Failed tasks** - Only errors
3. **Specific deployment** - Tasks for one deployment
4. **Specific task** - Details about a known task ID

### Step 2: Query Tasks

**Recent tasks (all states):**
```
bosh_tasks(limit: 10)
```

**Failed tasks only:**
```
bosh_tasks(state: "error", limit: 10)
```

**Tasks for specific deployment:**
```
bosh_tasks(deployment: "<deployment>", limit: 10)
```

**Currently running tasks:**
```
bosh_tasks(state: "processing")
```

### Step 3: Present Task List

Format clearly:
```
Recent BOSH Tasks
=================

ID    State       Deployment  Description                  Time
----------------------------------------------------------------------
1234  done        cf          update deployment            2 hours ago
1233  done        cf          run errand smoke_tests       4 hours ago
1232  error       cf          run errand acceptance_tests  6 hours ago
1231  done        redis       create deployment            1 day ago
1230  done        mysql       snapshot deployment          1 day ago
```

### Step 4: Investigate Specific Task

If the operator asks about a task or wants to see errors:

```
bosh_task(id: <task-id>, output: true)
```

Present the details:
```
Task 1232 Details
=================

ID:          1232
State:       error
Description: run errand acceptance_tests
Deployment:  cf
User:        admin
Started:     2024-11-26 10:00:00
Duration:    15 minutes

Result:
Error: Errand acceptance_tests failed
  - Router connectivity test failed
  - Expected 200, got 503

Output (last 50 lines):
[stdout] Running test: router_connectivity
[stdout] Sending request to https://api.cf.example.com
[stderr] Error: Connection refused
[stdout] Test FAILED
```

### Step 5: Analyze Failure Patterns

For error tasks, help identify the root cause:

**Deployment failures:**
```
Common causes:
- Resource quota exceeded (IaaS)
- Network connectivity issues
- Invalid manifest configuration
- Stemcell/release not found

Suggested actions:
1. Check IaaS quotas and limits
2. Verify network configuration
3. Review manifest changes
4. Ensure required stemcells/releases are uploaded
```

**Errand failures:**
```
Common causes:
- Dependent services not ready
- Configuration errors
- Timeout issues
- Test environment problems

Suggested actions:
1. Check if dependent deployments are healthy
2. Review errand logs for specific errors
3. Verify connectivity to external services
4. Consider running with different parameters
```

**VM operation failures:**
```
Common causes:
- IaaS API errors
- Resource exhaustion
- Agent communication issues
- Persistent disk problems

Suggested actions:
1. Check IaaS console for VM status
2. Verify BOSH agent logs
3. Check disk space and quotas
4. Review CPI logs for details
```

### Step 6: Wait for Running Tasks

If there are processing tasks:

```
bosh_task_wait(id: <task-id>, timeout: 600)
```

Report progress:
```
Waiting for task 1235...
Current state: processing
Description: update deployment cf

This task is still running. Would you like me to:
1. Continue waiting (up to 10 minutes)
2. Check back later
3. View current task output
```

### Step 7: Suggest Next Steps

Based on findings:

**If errors found:**
```
Found 2 failed tasks in the last 24 hours.

Task 1232 (acceptance_tests): Router connectivity issues
Task 1228 (update deployment): Resource quota exceeded

Would you like me to:
1. Get detailed output from these tasks
2. Check deployment health
3. Help troubleshoot the underlying issues
```

**If all tasks successful:**
```
No failed tasks in the last 24 hours.

Recent activity:
- 5 successful deployments
- 2 successful errands
- 3 successful VM operations

Everything looks normal. Is there a specific operation you want to review?
```

## Quick Task Queries

**What failed recently?**
```
bosh_tasks(state: "error", limit: 5)
```

**What's running now?**
```
bosh_tasks(state: "processing")
```

**What happened to deployment X?**
```
bosh_tasks(deployment: "<deployment>", limit: 10)
```

**Show me task 1234:**
```
bosh_task(id: 1234, output: true)
```

## Task States

Explain task states if asked:

```
BOSH Task States:
-----------------
queued      - Task waiting to run
processing  - Task currently executing
done        - Task completed successfully
error       - Task failed
cancelled   - Task was cancelled by user
timeout     - Task exceeded time limit
```

## Correlation with Other Skills

Link to other skills when relevant:

- **Failed VM operations**: Suggest `troubleshoot-vm`
- **Failed deployments**: Recommend reviewing manifest changes
- **Unhealthy after task**: Suggest `deployment-health`
- **Need to retry operation**: Suggest `restart-deployment` or relevant skill
