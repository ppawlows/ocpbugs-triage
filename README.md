# OCPBUGS Triaging Automation

Automated triaging system for OpenShift bug reports that analyzes, categorizes, and routes OCPBUGS issues to appropriate engineering components.

## Quick Start

Run the automation:
```bash
cat prompt.md | claude
```

## What It Does

- Queries JIRA for untriaged OCPBUGS issues
- Analyzes technical content to determine appropriate component assignments
- Generates HTML reports with triaging decisions and rationale
- Provides component assignment recommendations (never auto-applies changes)

## Files

- `prompt.md` - Main automation prompt
- `CLAUDE.md` - Detailed project instructions
- `ocpbugs_components_reference.html` - Official component list
- `triage_workflow.md` - Process documentation