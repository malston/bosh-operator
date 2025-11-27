---
name: setup-bosh-mcp
description: Use when BOSH tools are not available, when the user asks about connecting to BOSH, or when any bosh_* tool call fails with "tool not found". Guides operators through configuring the bosh-mcp-server connection.
---

# Setup BOSH MCP Server

You are helping an operator connect Claude Code to their BOSH Director via the bosh-mcp-server.

## First: Check if Already Configured

Before guiding setup, check if BOSH tools are available by looking for `bosh_deployments` in your available tools. If it exists and works, tell the operator they're already connected and offer to run a health check.

## Setup Workflow

### Step 1: Determine Authentication Method

Ask the operator how they authenticate to BOSH:

1. **Direct BOSH credentials** - They have a client ID and secret for the BOSH Director
2. **Ops Manager** - They use `om bosh-env` to get credentials
3. **BOSH CLI config** - They have credentials in `~/.bosh/config`

### Step 2: Collect Credentials

Based on their answer:

**For Direct BOSH credentials**, ask for:
- `BOSH_ENVIRONMENT` - The BOSH Director URL (e.g., `https://10.0.0.5:25555`)
- `BOSH_CLIENT` - The UAA client name
- `BOSH_CLIENT_SECRET` - The UAA client secret
- `BOSH_CA_CERT` - Path to the CA certificate file (or the cert content itself)

**For Ops Manager**, ask for:
- `OM_TARGET` - The Ops Manager URL (e.g., `https://opsman.example.com`)
- `OM_USERNAME` - Ops Manager admin username
- `OM_PASSWORD` - Ops Manager admin password
- Note: Credentials are cached for 5 minutes

**For BOSH CLI config**, explain that no additional setup is needed if `~/.bosh/config` exists with valid credentials.

### Step 3: Get the bosh-mcp-server Binary

Ask if they have the bosh-mcp-server binary installed. If not, provide options:

```bash
# Option 1: Download from releases
# Visit https://github.com/malston/bosh-mcp-server/releases

# Option 2: Build from source
go install github.com/malston/bosh-mcp-server/cmd/bosh-mcp-server@latest
```

### Step 4: Generate MCP Configuration

Create the `.claude/mcp.json` file in their project directory:

```json
{
  "mcpServers": {
    "bosh": {
      "command": "/path/to/bosh-mcp-server",
      "env": {
        "BOSH_ENVIRONMENT": "<collected-value>",
        "BOSH_CLIENT": "<collected-value>",
        "BOSH_CLIENT_SECRET": "<collected-value>",
        "BOSH_CA_CERT": "<collected-value>"
      }
    }
  }
}
```

Or for Ops Manager:

```json
{
  "mcpServers": {
    "bosh": {
      "command": "/path/to/bosh-mcp-server",
      "env": {
        "OM_TARGET": "<collected-value>",
        "OM_USERNAME": "<collected-value>",
        "OM_PASSWORD": "<collected-value>"
      }
    }
  }
}
```

### Step 5: Security Warning

Warn the operator:
- Avoid committing secrets to version control
- Consider using environment variables instead of hardcoding in mcp.json
- The bosh-mcp-server requires confirmation tokens for destructive operations

### Step 6: Verify Connection

After the operator restarts Claude Code:
1. Try calling `bosh_deployments` to verify connectivity
2. If successful, offer to run the `deployment-health` skill
3. If failed, help troubleshoot (check URL, credentials, network)

## Troubleshooting

Common issues:
- **Certificate errors**: Ensure BOSH_CA_CERT points to the correct CA certificate
- **Connection refused**: Verify the BOSH Director URL is accessible
- **Authentication failed**: Double-check client credentials
- **om command not found**: Ensure the `om` CLI is installed and in PATH
