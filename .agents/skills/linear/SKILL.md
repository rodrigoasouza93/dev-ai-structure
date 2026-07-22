---
name: linear
description: Manage issues, projects & team workflows in Linear. Use when the user wants to read, create or update tickets through a connected Linear MCP server.
---

# Linear

## Overview

This skill provides a structured workflow for managing issues, projects, and team workflows through the Linear MCP server connected to the active AI client.

## Prerequisites

- The Linear MCP server must be configured in the active client's MCP settings and accessible via OAuth.
- If MCP tools are unavailable, ask the user to add the Linear MCP server using the URL `https://mcp.linear.app/mcp`, then reconnect.
- Confirm access to the relevant Linear workspace, teams, and projects before starting.

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1

Clarify the user's goal and scope (e.g., issue triage, sprint planning, documentation audit, workload balance). Confirm team/project, priority, labels, cycle, and due dates as needed.

### Step 2

Select the appropriate workflow (see Practical Workflows below) and identify the Linear MCP tools you will need. Confirm required identifiers (issue ID, project ID, team key) before calling tools.

### Step 3

Execute Linear MCP tool calls in logical batches:

- Read first (list/get/search) to build context.
- Create or update next (issues, projects, labels, comments) with all required fields.
- For bulk operations, explain the grouping logic before applying changes.

### Step 4

Summarize results, call out remaining gaps or blockers, and propose next actions (additional issues, label changes, assignments, or follow-up comments).

## Available Tools

Tool names and prefixes vary by client. Discover the connected Linear MCP tools instead of assuming a fixed prefix.

**Issue Management:** `get_issue`, `list_issues`, `save_issue`, `get_issue_status`, `list_issue_statuses`, `list_issue_labels`, `create_issue_label`

**Project & Team:** `get_project`, `list_projects`, `save_project`, `get_team`, `list_teams`, `get_user`, `list_users`, `get_milestone`, `list_milestones`, `save_milestone`

**Documentation & Collaboration:** `get_document`, `list_documents`, `save_document`, `list_comments`, `save_comment`, `delete_comment`, `list_cycles`

**Initiatives & Status:** `get_initiative`, `list_initiatives`, `save_initiative`, `get_status_updates`, `save_status_update`, `delete_status_update`

**Attachments & Diffs:** `get_attachment`, `list_diffs`, `get_diff`, `get_diff_threads`, `extract_images`

## Practical Workflows

- **Sprint Planning:** Review open issues for a target team, pick top items by priority, and assign to a cycle.
- **Bug Triage:** List critical/high-priority bugs, rank by user impact, and move the top items to "In Progress."
- **Documentation Audit:** Search documentation, then open labeled "documentation" issues for gaps or outdated sections.
- **Team Workload Balance:** Group active issues by assignee, flag high load, and suggest redistributions.
- **Release Planning:** Create a project with milestones and generate issues with estimates.
- **Cross-Project Dependencies:** Find all "blocked" issues, identify blockers, and create linked issues if missing.
- **Automated Status Updates:** Find stale issues and add status comments based on current state.
- **Smart Labeling:** Analyze unlabeled issues, suggest/apply labels, and create missing label categories.

## Tips

- Batch operations for related changes.
- Confirm required identifiers (issue ID, team key) before calling create/update tools.
- Break large updates into smaller batches to avoid rate limits.

## Troubleshooting

- **MCP not responding:** Verify the Linear MCP server is enabled in the active client's settings and the OAuth session is active.
- **Missing data:** Check workspace permissions and confirm the correct team/project is selected.
- **Tool call errors:** Provide all required fields; split complex bulk requests into smaller operations.
