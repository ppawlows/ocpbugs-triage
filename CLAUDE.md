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
- `claude.md` - This file - project guidance and instructions
- `ocpbugs_components_reference.html` - Verified OCPBUGS component list from official project
- `ocpbugs_security_final_report.html` - Example security issues component assignment report
- `triage_workflow.md` - Detailed triaging process documentation

## Triaging Decision Logic

### Component Assignment Principles
1. **OpenShift Architecture Knowledge**: Understand where issues fit in OpenShift's technical architecture
2. **Component Description Matching**: Align technical contexts with official component descriptions from CSV
3. **Technical Context Over Keywords**: Analyze the underlying technical problem, not just keyword presence
4. **Technical Specificity**: Assign to the most specific component that handles the issue
5. **Ownership Clarity**: Route to components with clear engineering team ownership
6. **Functional Boundaries**: Respect component functional responsibilities
7. **Escalation Path**: Use generic components (Security, Storage) only when specific assignment unclear


## Jira CLI Commands

Key commands for OCPBUGS triaging:
- Query untriaged issues: `jira issue list -q 'project = OCPBUGS AND component is EMPTY AND status in (NEW, POST)' --plain --no-headers --columns key,summary,status,priority`
- View issue details: `jira issue view OCPBUGS-12345`
- Set component (should never be executed): `jira issue edit OCPBUGS-12345 --custom component="Component Name" --no-input`
- Query by component: `jira issue list -q 'project = OCPBUGS AND component = "Component Name"' --plain --no-headers`

Important constraints:
- Run maximum 2 jira CLI commands in parallel to avoid rate limiting
- Use `--no-input` flag to avoid interactive prompts
- Use `--plain --no-headers` for scriptable output
- Component names must match exactly (case-sensitive)
- Never change JIRA tickets content

## Automation Workflow

To run the OCPBUGS triaging automation:
```bash
cat prompt.md | claude
```

The automation will:
1. Query for untriaged issues
2. Analyze each issue's technical content
3. Determine appropriate component assignments
4. Generate HTML report with recommendations
5. Optionally apply component assignments (if requested)

## Analysis Methodology

### Issue Content Analysis
- **Summary parsing**: Extract key terms, error messages, affected services
- **Description analysis**: Identify technical details, stack traces, logs
- **Component inference**: Map technical details to appropriate OCPBUGS components

### Quality Assurance
- **Component verification**: Ensure assigned components exist in OCPBUGS
- **Cross-reference validation**: Check similar historical issue assignments
- **Rationale documentation**: Provide clear reasoning for each assignment decision

## Important Notes

### Security and Compliance
- Never modify JIRA issues without explicit user permission
- Always generate reports before making any changes
- Respect component ownership and engineering team boundaries
- Follow Red Hat security practices for issue handling

### Error Handling
- Validate component names against official OCPBUGS component list
- Handle rate limiting gracefully with appropriate delays
- Provide fallback assignments for unclear cases
- Log all triaging decisions with detailed rationale

### Performance Optimization
- Batch process multiple issues efficiently
- Cache component lists to reduce API calls
- Do not parallel processing
- Generate comprehensive reports for audit trails

## Usage Examples

### Basic Triaging
```bash
# Query untriaged security issues
jira issue list -q 'project = OCPBUGS AND component is EMPTY AND labels in (Security)'

# Analyze and assign specific issue
jira issue view OCPBUGS-12345
```

### Bulk Triaging
```bash
# Process all untriaged issues from last week
jira issue list -q 'project = OCPBUGS AND component is EMPTY AND created >= -7d'
# ... batch analysis and assignment ...
```

This automation system is designed to improve OCPBUGS triaging efficiency while maintaining accuracy and proper engineering team routing.