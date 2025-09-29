# OCPBUGS Triaging Automation

Automated triaging system for OpenShift bug reports that analyzes, categorizes, and routes OCPBUGS issues to appropriate engineering components.

## Prerequisites

### JIRA CLI Setup

1. **Install JIRA CLI** (if not already installed):
   ```bash
   # macOS with Homebrew
   brew install jira-cli

   # Or download from: https://github.com/ankitpokhrel/jira-cli/releases
   ```

2. **Configure JIRA CLI Authentication**:
   ```bash
   # Set up your API token (required before running claude)
   export JIRA_API_TOKEN="your-jira-api-token-here"

   # Initialize JIRA CLI configuration
   jira init
   ```

3. **JIRA Init Configuration** (when prompted):
   - **Installation type**: `local` (for Red Hat JIRA)
   - **Server**: `https://issues.redhat.com`
   - **Login**: Your Red Hat email address
   - **Auth type**: `bearer` (uses JIRA_API_TOKEN)
   - **Project**: `OCPBUGS` (default project for queries)
   - **Board**: Leave empty or specify your preferred board

4. **Get Your JIRA API Token**:
   - Go to [Red Hat JIRA](https://issues.redhat.com)
   - Click on your profile (top right corner)
   - Navigate to: **Personal Access Tokens**
   - Click **Create token**
   - **Important**: For security reasons, set token expiration to **maximum 90 days**
   - Copy the generated token and set it in your environment: `export JIRA_API_TOKEN="your-token"`

5. **Verify Configuration**:
   ```bash
   # Test JIRA CLI access
   jira issue list -q "project = OCPBUGS" --limit 1
   ```

## Quick Start

Run the automation:
```bash
# Make sure JIRA_API_TOKEN is set
export JIRA_API_TOKEN="your-jira-api-token"

# Run the triaging automation
cat prompt.md | claude
```

## What It Does

- Queries JIRA for untriaged OCPBUGS issues
- Analyzes technical content to determine appropriate component assignments
- Generates HTML reports with triaging decisions and rationale
- Provides component assignment recommendations (never auto-applies changes)