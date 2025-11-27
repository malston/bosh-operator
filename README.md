# bosh-operator

A Claude Code plugin that provides a conversational BOSH operations assistant.

## Overview

This plugin helps operators manage BOSH deployments through natural language conversations. It provides workflow-based skills that guide you through common operational tasks with built-in best practices and safety guardrails.

## Prerequisites

- [Claude Code](https://claude.ai/download) installed
- [bosh-mcp-server](https://github.com/malston/bosh-mcp-server) for BOSH connectivity

## Installation

```bash
claude plugin install github:malston/bosh-operator
```

Or add to your Claude Code plugins manually.

## Skills

### setup-bosh-mcp

Guides you through connecting Claude Code to your BOSH Director.

**Trigger phrases:**
- "Connect to BOSH"
- "Setup BOSH"
- "Configure BOSH MCP"

### troubleshoot-vm

Diagnoses and fixes unhealthy VMs in your deployments.

**Trigger phrases:**
- "A VM is failing"
- "Help me troubleshoot diego_cell/2"
- "Why is my router unhealthy?"

### restart-deployment

Safe rolling restarts with pre-flight checks and verification.

**Trigger phrases:**
- "Restart the redis deployment"
- "I need to restart all routers"
- "Rolling restart of cf"

### deployment-health

Comprehensive health reports for your BOSH infrastructure.

**Trigger phrases:**
- "Is my cf deployment healthy?"
- "Show me deployment status"
- "How's my BOSH infrastructure?"

### scale-deployment

Guidance for scaling operations (note: actual scaling requires manifest changes).

**Trigger phrases:**
- "How do I scale diego_cells?"
- "Show me instance counts"
- "Stop some instances temporarily"

### check-tasks

Review task history and investigate failures.

**Trigger phrases:**
- "What tasks failed today?"
- "Show me recent BOSH activity"
- "What happened to task 1234?"

## Example Conversations

**Health check:**
```
You: How's my CF deployment?

Claude: Let me check the health of your CF deployment...
[Uses deployment-health skill to gather and present status]
```

**Troubleshooting:**
```
You: Diego_cell/2 is having issues

Claude: I'll help you diagnose diego_cell/2. Let me check its status...
[Uses troubleshoot-vm skill to diagnose and recommend fixes]
```

**Restart:**
```
You: Restart the routers

Claude: I'll help you restart the router instances safely. Let me first check the current state...
[Uses restart-deployment skill with pre-flight checks]
```

## Safety Features

- **Confirmation tokens**: Destructive operations require explicit confirmation
- **Pre-flight checks**: Skills verify deployment health before operations
- **Lock checking**: Operations check for active locks to prevent conflicts
- **Best practices**: Skills recommend safe approaches (AZ-by-AZ restarts, etc.)

## Testing

You can test this plugin safely using the [bosh-mock-director](https://github.com/malston/bosh-mock-director) which simulates a BOSH environment without requiring real infrastructure.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

MIT
