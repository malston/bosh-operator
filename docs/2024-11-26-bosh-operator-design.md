# BOSH Operator Plugin Design

A Claude Code plugin that provides a conversational BOSH operations assistant for operators managing BOSH deployments.

## Overview

The plugin provides workflow-based skills that guide operators through common BOSH tasks using natural language. It leverages the `bosh-mcp-server` for actual BOSH connectivity.

## Goals

- Help operators troubleshoot, manage, and monitor BOSH deployments conversationally
- Encode operational best practices and safety guardrails into skills
- Detect bosh-mcp-server configuration and guide setup if needed
- Support troubleshooting, day-2 operations, and visibility/reporting

## Plugin Structure

```
bosh-operator/
├── package.json
├── skills/
│   ├── setup-bosh-mcp.md
│   ├── troubleshoot-vm.md
│   ├── restart-deployment.md
│   ├── deployment-health.md
│   ├── scale-deployment.md
│   └── check-tasks.md
├── docs/
│   └── 2024-11-26-bosh-operator-design.md
└── README.md
```

## Skills

### setup-bosh-mcp

Detects if bosh-mcp-server is configured and guides operators through setup.

**Workflow:**
1. Check if `bosh_*` MCP tools are available
2. Ask about authentication method (direct credentials, Ops Manager, ~/.bosh/config)
3. Collect required credentials
4. Generate `.claude/mcp.json` configuration
5. Verify connection after restart

### troubleshoot-vm

Diagnoses and fixes unhealthy VMs.

**Workflow:**
1. Identify deployment and unhealthy VMs via `bosh_vms`
2. Get detailed process state via `bosh_instances`
3. Check recent failures via `bosh_tasks`
4. Recommend action (restart, recreate) based on diagnosis
5. Execute with confirmation and verify fix

**Safety guardrails:**
- Never recreate multiple VMs without explicit approval
- Warn about bootstrap instances
- Check locks before operations

### restart-deployment

Safe rolling restart with pre/post checks.

**Workflow:**
1. Pre-flight: check health, locks, confirm scope
2. Warn about disruption and recommend timing
3. Execute restart with confirmation token
4. Verify all instances recovered

**Best practices:**
- Restart one AZ at a time for HA
- Warn about singleton jobs
- Recommend off-peak timing

### deployment-health

Comprehensive health visibility and reporting.

**Workflow:**
1. Gather data from deployments, VMs, instances, tasks
2. Categorize as Healthy/Degraded/Critical
3. Present summary with recommendations
4. Offer drill-down to troubleshooting

**Additional views:**
- Stemcell currency
- Release versions

### scale-deployment

Advisory skill for scaling operations.

**Scope:**
- Explain that BOSH scaling requires manifest changes
- Help identify current instance counts
- Support temporary capacity reduction via stop/start
- Warn about implications

### check-tasks

Task history and failure investigation.

**Workflow:**
1. Query tasks with filters (state, deployment, limit)
2. Get task details and output
3. Summarize failures and suggest remediation

## Cross-Skill Integration

- All skills check for MCP connectivity, suggest `setup-bosh-mcp` if missing
- `deployment-health` suggests `troubleshoot-vm` when issues found
- `troubleshoot-vm` suggests `restart-deployment` or `check-tasks` as needed

## Testing

The plugin can be tested safely using `bosh-mock-director` which simulates a real BOSH environment with sample deployments.

## Dependencies

- `bosh-mcp-server` - Provides the `bosh_*` MCP tools
- Claude Code - Plugin runtime environment
