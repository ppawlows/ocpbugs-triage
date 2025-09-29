# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with OCPBUGS triaging automation in this repository.

## Project Overview

This is an OCPBUGS automated triaging system that helps analyze, categorize, and route OpenShift bug reports to appropriate engineering components. The system focuses on improving issue assignment accuracy and reducing manual triaging overhead.

## Core Workflow

The primary OCPBUGS triaging workflow involves:
1. Querying JIRA for OCPBUGS issues requiring triage using `jira issue list` with JQL
2. Analyzing issue content, summaries, and descriptions to determine appropriate component assignment
3. Extracting technical details to identify affected OpenShift subsystems
4. Generating component assignment recommendations based on issue analysis
5. Creating detailed HTML reports with triaging decisions and rationale
6. It will never update JIRA cards, but may provide in the HTML report commands to do so

## Key Files

- `prompt.md` - Main prompt template for OCPBUGS triaging automation
- `CLAUDE.md` - This file - project guidance and instructions
- `ocpbugs_components.csv` - Official OCPBUGS component list with descriptions
- `triage_workflow.md` - Detailed triaging process documentation

## Component Assignment Principles

1. **OpenShift Architecture Knowledge**: Understand where issues fit in OpenShift's technical architecture
2. **Component Description Matching**: Align technical contexts with official component descriptions from CSV
3. **Technical Context Over Keywords**: Analyze the underlying technical problem, not just keyword presence
4. **Technical Specificity**: Assign to the most specific component that handles the issue
5. **Ownership Clarity**: Route to components with clear engineering team ownership
6. **Functional Boundaries**: Respect component functional responsibilities
7. **Escalation Path**: Use generic components (Security, Storage) only when specific assignment unclear

## Critical Constraints

- **READ-ONLY OPERATIONS**: Never modify JIRA issues - automation only suggests assignments
- **Mandatory HTML Report**: Generate `triaging_report_YYYYMMDD.html` on every run
- **Rate Limiting**: Run maximum 2 JIRA CLI commands in parallel
- **Component Validation**: All assignments must exist in `ocpbugs_components.csv`
- **No Automatic Execution**: All JIRA changes require manual approval

## Automation Entry Point

To run the OCPBUGS triaging automation:
```bash
cat prompt.md | claude
```

The automation targets issues assigned to generic "Security" component that need specific assignment using the query:
```
project = OCPBUGS AND status in (NEW, POST, ASSIGNED) AND component = security
```

This read-only automation system improves OCPBUGS triaging efficiency by generating detailed HTML reports with component assignment recommendations while maintaining accuracy and proper engineering team routing.