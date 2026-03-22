---
name: swa-triage-sweep
description: Sweep the SWA Linear triage backlog, propose priorities and project assignments for each ticket, publish a Slack Canvas with a full table, and post an announcement to #swaps-leads. Use when the user says "triage sweep", "sweep triage", "review triage", or "run triage" for the Swaps team.
user_invocable: true
version: 1.0.0
---

# SWA Triage Sweep

Fetch all Triage tickets from the SWA team in Linear, propose a priority, status, and project for each one, publish a Slack Canvas with a formatted table, and post a summary announcement to #swaps-leads.

## Step 1: Fetch Triage tickets and active projects

Call both in parallel:

- `linear_list_issues` with `team: "SWA"`, `state: "triage"`, `limit: 250`
- `linear_list_projects` with `team: "SWA"`, `limit: 50`

## Step 2: Classify each ticket

For each ticket, propose three things:

### Priority (use Linear's scale)

| Priority | When to apply |
|---|---|
| **Urgent** | Production impact, data loss, active user-facing failures |
| **High** | This-week work: bugs with user/partner impact, CS escalations needing human response |
| **Medium** | Next sprint: partner feature requests, security vulns (medium CVSS), unblocked dep upgrades |
| **Low** | Backlog: code quality, cleanup, docs, minor bugs, low-severity vulns |

### Status

| Status | When to apply |
|---|---|
| **Todo** | Should be picked up immediately or this sprint |
| **Backlog** | Valid work, but not time-sensitive |

### Project

Match each ticket to the most relevant active SWA project using these rules:

- **502 / startup / deployment bugs** → Swaps.xyz Maintenance
- **Transaction lifecycle, pending/failed state, txsv2 data loss** → Swap Lifecycle Tracking Quality
- **SOL routing, Solana-specific bugs** → Solana Optimizations
- **CS escalations (untraceable tx, refund requests, status inquiries)** → No project (CS Sweep)
- **KYC/AML, ChangeNow, partner integrations** → Partner Pipeline
- **Security vulnerabilities (Socketdev label, CVSS scores)** → Vulnerability Reports
- **Dep update rebasing / shipping** → Vulnerability Reports
- **Fee display, fee accuracy, fee reconciliation** → Fee Reconciliation
- **DB schema, camelCase migrations, Mongo cleanup** → Database Cleanup + Migration
- **TypeScript strictness (strictNullChecks, noImplicitAny)** → Strictify box-monorepo
- **Request retries, third-party request standardisation** → Request Service
- **Docs, dead code, console UX, naming, general cleanup** → Swaps.xyz Maintenance
- **No clear match** → Swaps.xyz Maintenance (catch-all)

> CS escalations are support tickets from CS/Ben Nock/Alberto — they need a quick human investigation and close, not an engineering project. Group them together.

## Step 3: Create the Slack Canvas

Call `slack_create_canvas` with:

- **title**: `SWA Triage Backlog — [today's date]`
- **content**: a markdown table with these columns:

```
| Ticket | Title | Priority | Status | Project |
```

Rows ordered by priority (Urgent → High → Medium → Low), then by ticket number within each group.

- Link each ticket cell: `[SWA-XXXX](https://linear.app/moonpay/issue/SWA-XXXX/...)`
- Link each project cell to its Linear project URL
- For CS Sweep tickets, use `CS Sweep` (no link) in the Project column
- Use emoji for priority: 🚨 Urgent, 🔴 High, 🟡 Medium, 🟢 Low
- Escape any literal `|` characters in cell content with `\|`

Add a header paragraph above the table:

```
Generated [DATE] · [N] tickets reviewed · Full Linear doc: [link if one exists]
```

## Step 4: Post announcement to #swaps-leads

Channel ID: `C0AKBT8J6VC`

Call `slack_send_message` with a message in this format:

```
📋 *SWA Triage Sweep — [DATE]*
[N] tickets reviewed and mapped to projects. Full table in the Canvas below.

🚨 *Urgent*
• [SWA-XXXX](url) Title
  Proposing | Status: Todo | Project: [Project](url)

🔴 *High*
• [SWA-XXXX](url) Title
  Proposing | Status: Todo | Project: [Project](url)
[... one bullet per ticket or grouped if same project/pattern ...]

🟡 *Medium*
[...]

🟢 *Low*
[...]

👉 [Canvas: SWA Triage Backlog](canvas_url)
```

**Formatting rules:**
- Title and priority headers must be bold: `*text*` — place emoji *outside* the `*` markers (e.g. `🔴 *High*` not `*🔴 High*`) to prevent Slack from breaking bold rendering
- Use markdown links `[text](url)` — do NOT use `<url|text>` angle-bracket syntax (the MCP URL-encodes the `|`)
- If the message would exceed ~4500 characters, split into two sequential messages (Urgent+High in first, Medium+Low in second)
- CS escalation tickets can be grouped on one line: `SWA-XXXX, SWA-YYYY, SWA-ZZZZ — CS escalations, quick look and close`

## Step 5: Report back

After sending, return:
- Canvas URL
- Slack message URL
- Count of tickets processed by priority tier

## Prerequisites

- Linear MCP connected (for fetching triage issues and projects)
- Slack MCP connected (for Canvas creation and message posting)

## Graceful Degradation

| Tool | If unavailable |
|---|---|
| Linear MCP | Inform user — cannot proceed without ticket data |
| Slack MCP | Output the table as markdown in the conversation instead |
| Canvas creation fails | Fall back to posting the table as two Slack messages |
