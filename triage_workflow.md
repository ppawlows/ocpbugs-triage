# OCPBUGS Triaging Workflow Documentation

## Overview

This document describes what happens when Claude Code executes the OCPBUGS triaging automation via `cat prompt.md | claude`.

**⚠️ IMPORTANT:** The automation is READ-ONLY. Claude never modifies JIRA issues directly.

## Automation Execution Flow

### Step 1: Claude Reads Instructions
When `cat prompt.md | claude` is executed:
- Claude loads the triaging instructions from prompt.md
- Claude loads the component definitions from ocpbugs_components.csv
- Claude understands the target query and analysis requirements

### Step 2: Claude Queries JIRA (READ-ONLY)
Claude executes the target query:
```
project = OCPBUGS AND status in (NEW, POST, ASSIGNED) AND component = security
```
This retrieves issues currently assigned to the generic "Security" component that need specific assignment.

### Step 3: Claude Analyzes Each Issue (READ-ONLY)
For each issue found, Claude:
1. **Reads issue details** using `jira issue view ISSUE-KEY`
2. **Extracts CVE ID** from issue summary if present (e.g., CVE-2024-1234)
3. **Fetches CVE details** using WebFetch from authoritative sources:
   - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-YYYY-NNNN
   - https://nvd.nist.gov/vuln/detail/CVE-YYYY-NNNN
4. **Analyzes CVE technical context** including affected software components and vulnerability impact
5. **Extracts technical content** from summary, description, and CVE details
6. **Identifies keywords** related to OpenShift components (systemd, build, networking, etc.)


### Step 4: Claude Researches Historical Patterns (READ-ONLY)
Claude analyzes similar resolved issues to learn assignment patterns:
- **Research similar issues**: Searches for comparable technical contexts in already-triaged OCPBUGS using queries like:
```
project = OCPBUGS AND text ~ $TECHNICAL_CONTEXT AND component != security
```
- **Analyze OpenShift knowledge**: Reviews how similar technical problems were previously assigned based on OpenShift architecture understanding
- **Validate against component descriptions**: Ensures assignments align with official component responsibilities

### Step 5: Claude Assigns Issues to Components
Claude applies OpenShift knowledge-based component assignment:
1. **OpenShift Architecture Analysis**: Understands where the technical issue fits in OpenShift's architecture
2. **Component Responsibility Mapping**: Matches issue context to component descriptions from CSV
3. **Avoids Simple Keyword Matching**: Goes beyond keywords to understand technical relationships
4. **Applies Specificity Logic**: Chooses the most specific component that owns the functionality
5. **Validates Against Historical Patterns**: Cross-references with learned patterns from previous reports

### Step 6: Claude Generates Required Files (MANDATORY)
**HARD REQUIREMENT**: Claude MUST generate these files on every run:

**1. HTML Triaging Report** (`triaging_report_YYYYMMDD.html`):
- **Executive summary** of issues processed
- **Detailed analysis table** with current vs. recommended components including:
  - Direct JIRA links to each analyzed issue
  - CVE ID and links to CVE database details (if present)
  - CVE technical summary and affected components
  - Detailed technical rationale for each assignment decision incorporating CVE analysis
  - Research sources and links to similar issues used for decision-making
  - Confidence levels based on research evidence strength and CVE context
- **Research documentation** section documenting the analysis process:
  - CVE database sources and links used for vulnerability analysis
  - JIRA queries used to find similar historical issues
  - Links to specific source issues that informed decisions
  - Clear explanations of how CVE analysis and research sources relate to assignment decisions
- **Component statistics** and usage patterns
- **Quality metrics** and confidence levels
- **Suggested JIRA commands** for manual execution


The HTML report is the mandatory output that enables analysis review and provides assignment recommendations.

## What Claude Does NOT Do

- ❌ **No JIRA modifications**: Claude never runs `jira issue edit` commands
- ❌ **No component assignments**: Claude only suggests assignments
- ❌ **No status changes**: Claude doesn't modify issue status or priority
- ❌ **No automatic execution**: All JIRA changes require manual approval

## Typical Automation Session

### Input
```bash
cd ~/projects/ocpbugs_triage
cat prompt.md | claude
```

### Claude's Process
1. **Loads configuration** from prompt.md and ocpbugs_components.csv
2. **Queries security issues**
3. **Checks similar OCPBUGS** that have been triaged already
4. **Analyzes each issue** for component assignment
5. **Generates HTML report** saved as `triaging_report_YYYYMMDD.html`
6. **Outputs summary** to terminal

### Output Files Generated (MANDATORY)
**REQUIRED on every run:**
- `triaging_report_YYYYMMDD.html` - Comprehensive triaging analysis report with component assignments, technical rationale, and suggested JIRA commands

## Claude's Decision-Making Process

### Component Assignment Logic
Claude follows this OpenShift knowledge-based hierarchy:
1. **OpenShift Architecture Understanding**: Analyze where the issue fits in OpenShift's technical stack
2. **Component Responsibility Analysis**: Match issue context to official component descriptions in CSV
3. **Technical Context Over Keywords**: Understand the underlying technical problem, not just surface keywords
4. **Historical Pattern Validation**: Reference previous reports for consistency
5. **Most Specific Component**: Choose the component that directly owns the functionality
6. **Fallback Assignment**: Use broader components only when specific assignment unclear

### Example Analysis by Claude
**Issue**: "CVE-2025-4598 systemd: race condition allows local attacker to crash SUID program"

**Claude's Analysis Process**:
1. **CVE ID Extraction**: Identifies CVE-2025-4598 from issue summary
2. **CVE Database Research**: Fetches details from https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2025-4598
3. **CVE Technical Analysis**: systemd vulnerability affecting SUID programs and core dumps, allowing local privilege escalation
4. **Technical Context Understanding**: systemd service vulnerability on OpenShift nodes affecting system security
5. **OpenShift Architecture Analysis**: systemd runs on every OpenShift node and manages critical system services
6. **Component Responsibility Research**: Which component owns node-level system management in OpenShift?
7. **CSV Component Description**: Machine Config Operator "manages and applies configuration and updates of the base operating system and container runtime; essentially everything between the kernel and kubelet"
8. **Architecture Fit**: CVE shows systemd clearly falls between kernel and kubelet in the OpenShift stack
9. **Decision**: Assign to "Machine Config Operator"
10. **Rationale**: "Based on CVE-2025-4598 analysis showing systemd core functionality vulnerability, MCO owns systemd and all node-level system service management in OpenShift architecture"

### Quality Assurance by Claude
- **Component validation**: Ensures all suggested components exist in ocpbugs_components.csv
- **Consistency checking**: Similar issues get similar component assignments
- **Confidence scoring**: Claude provides confidence levels for each assignment
- **Manual review flags**: Complex cases are flagged for human review

## Running Different Workflows

### Basic Security Component Triage
```bash
cat prompt.md | claude
```
Claude processes current Security component issues.

### Historical Pattern Learning
```bash
./historical_learning.sh
```
Claude analyzes past assignments to improve future decisions (READ-ONLY).

### Custom Analysis
Modify the JQL query in prompt.md to target different issue sets, then run:
```bash
cat prompt.md | claude
```

## Expected Outcomes

### Successful Automation Run
- ✅ Issues queried and analyzed successfully
- ✅ HTML report generated with clear recommendations
- ✅ All analyzed issues include direct JIRA links
- ✅ All suggested components validated against CSV
- ✅ Technical rationale provided for each assignment with detailed explanations
- ✅ Research sources documented with links to supporting similar issues
- ✅ JIRA queries used for research are documented and reproducible
- ✅ Confidence levels assigned based on research evidence strength
- ✅ Ready-to-execute JIRA commands provided (for manual use)

### Manual Follow-up Required
After Claude completes the automation:
1. **Review the HTML report** for assignment accuracy
2. **Validate component recommendations** against engineering team ownership
3. **Execute suggested JIRA commands manually** if approved
4. **Monitor assignment feedback** for future improvement

## Integration with JIRA

### READ-ONLY Operations (Automated)
- `jira issue list` - Query issues
- `jira issue view` - Read issue details
- Pattern analysis and component mapping

### WRITE Operations (Manual Only)
- `jira issue edit` - Apply component assignments
- Issue prioritization and status updates
- Adding comments or labels

## Troubleshooting

### Common Issues
- **JQL syntax errors**: Claude will adjust query syntax automatically
- **Component validation failures**: Claude validates against CSV and reports errors
- **Rate limiting**: Claude respects JIRA API limits (max 2 concurrent requests)
- **Missing components**: Claude flags unknown components for manual review

### Performance Optimization
- Claude processes issues in reasonable batches (typically 20-50 issues)
- Historical pattern queries are limited to prevent performance issues

## Best Practices

### Before Running Automation
- Ensure ocpbugs_components.csv is up to date
- Review the target query in prompt.md for accuracy
- Check JIRA API connectivity

### After Automation Completes
- Review HTML report thoroughly before applying changes
- Validate component assignments with engineering teams
- Execute JIRA commands manually in small batches
- Monitor assignment feedback for pattern improvement

### Quality Control
- Claude provides confidence scores for each assignment based on research evidence
- Complex or uncertain cases are flagged for manual review
- All decisions include detailed technical rationale with supporting evidence
- Research sources are documented with direct links to similar JIRA issues
- JIRA queries used for research are saved and reproducible
- Assignment consistency is maintained across similar issues
- Decision-making process is fully transparent and auditable

This workflow ensures efficient, accurate OCPBUGS component assignment while maintaining full human control over JIRA modifications.