---
name: basecamp-summary
description: Pull Basecamp project data via CLI and generate an AI-analyzed daily summary report with hot topics, risk alerts, and key insights. Supports bilingual output (EN/CN), Feishu Wiki publishing, and HTML email delivery.
user_invocable: true
argument-hint: "[--timerange=7d] [--project=NAME] [--verbose] [--email] [--wiki] [--format=html|markdown]"
---

# Basecamp Daily Summary Generator

You are a project intelligence assistant. Use the **official Basecamp CLI** (`basecamp` command) to pull data and generate actionable daily summaries.

## Prerequisites & Auto-Setup

Before collecting data, **always check and auto-install** the Basecamp CLI if needed:

```bash
# Step 0: Check if basecamp CLI is available
which basecamp 2>/dev/null || ls ~/.local/bin/basecamp 2>/dev/null
```

**If NOT installed**, run this auto-install sequence:

```bash
# Detect OS and architecture
OS=$(uname -s | tr '[:upper:]' '[:lower:]')   # linux or darwin
ARCH=$(uname -m)                                # x86_64 or arm64
case "$ARCH" in x86_64) ARCH="amd64" ;; aarch64) ARCH="arm64" ;; esac

# Fetch latest release download URL from GitHub
DOWNLOAD_URL=$(curl -fsSL https://api.github.com/repos/basecamp/basecamp-cli/releases/latest \
  | grep "browser_download_url" \
  | grep "${OS}_${ARCH}" \
  | head -1 \
  | cut -d '"' -f 4)

# Download and install
mkdir -p ~/.local/bin
curl -fsSL "$DOWNLOAD_URL" -o /tmp/basecamp-cli.tar.gz
tar -xzf /tmp/basecamp-cli.tar.gz -C ~/.local/bin/ basecamp
chmod +x ~/.local/bin/basecamp
export PATH="$HOME/.local/bin:$PATH"

# Verify
basecamp --version
```

**If installed but not authenticated**, run:

```bash
basecamp auth login
# → Opens browser for OAuth authorization
# → User selects their Basecamp account
```

**Check auth status:**

```bash
basecamp auth status
```

## Workflow

### Phase 1: Data Collection (via Basecamp CLI)

Run these commands to collect data. Always use `--json --quiet` for machine-parseable output.

```bash
# 1. List all active projects
basecamp projects list --json --quiet

# 2. For each relevant project, collect recent activity
basecamp timeline --in <project_id> --json --quiet
basecamp messages list --in <project_id> --json --quiet
basecamp recordings list --type todo --in <project_id> --json --quiet

# 3. Cross-project reports
basecamp reports overdue --json --quiet
basecamp assignments list --json --quiet

# 4. Files (if relevant)
basecamp files list --in <project_id> --json --quiet

# 5. Attachments — download inline attachments when needed
basecamp messages show <id> --in <project_id> --download-attachments --json --quiet
```

**Filtering**: If `--project` is specified, filter projects by name regex. Otherwise, scan all accessible projects.

### Phase 2: AI Analysis & Report Generation

After collecting data, generate a report following the **Standard Report Structure** below.

#### Standard Report Structure

```
1. Title (H1)       — "Basecamp Daily Summary — YYYY-MM-DD"
2. Meta line         — Date, project count, method
3. Executive Summary — 2-3 sentence overview with key numbers bolded
4. Project Updates   — Per project:
   - Project info: ID | Client | Activity Level (High/Medium/Low)
   - Hot Topics (past 48 hours): numbered list with sub-bullets
   - Key messages, decisions, blockers
5. Risk Alerts       — Table format, color-coded by severity
6. Key Insights      — Bullet list of cross-project observations
7. Footer            — Generator info + Wiki link (if published)
```

#### Language Rules

Default: **Bilingual** (English primary, Chinese secondary)
- Section headers in English
- Content in English with Chinese annotations where helpful
- Override with `--format` parameter if specified

### Phase 3: Output

**Always save locally:**
- Markdown: `basecamp-reports/daily-3m-summary-{YYYY-MM-DD}.md`
- HTML (if `--email` or `--format=html`): `basecamp-reports/daily-3m-summary-{YYYY-MM-DD}.html`

**HTML Email Format** (when `--email` is used or for email delivery):

Follow this exact styling specification:
- Font: `Segoe UI, Arial, sans-serif`, color `#333`, line-height `1.6`, max-width `800px`
- H1 Title: `color:#1a73e8`, blue bottom border 2px
- Executive Summary H2: `color:#d32f2f` (red)
- Project Updates H2: `color:#1a73e8` (blue)
- Per project H3: with emoji prefix + project name
- Activity Level: colored `<span>` — green=High, yellow=Medium, red=Low
- Hot Topics: `<ol>` numbered list, each `<li><strong>Topic</strong> — context`, sub-bullets `<ul><li>` for details
- Risk Alerts: `<table>` with `border-collapse:collapse;width:100%`
  - High risk rows: `background:#ffcdd2`
  - Medium risk rows: `background:#fff3e0`
  - Cells: `padding:8px;border:1px solid #ddd`
- Key Insights: `<ul>` bullet list under blue H2
- Footer: `color:#999;font-size:12px`
- All inline styles (email-safe HTML, no external CSS)
- `<hr/>` separators between major sections
- People names always include affiliation: `Name (Company)`

Reference template: `templates/report-email.html` in this plugin directory.

**Optional — Feishu Wiki** (when `--wiki` is used):
- Create new doc under configured wiki node using `lark-cli docs +create --wiki-node <NODE>`
- Update index page with link to new report
- Requires: `lark-cli` installed and authenticated

**Optional — Email delivery** (when `--email` is used):
- Send via Composio Outlook MCP (`OUTLOOK_SEND_EMAIL`) if available
- Or via `lark-cli mail +send` if Feishu mail is configured
- Recipients: from plugin config `email_recipients`

### Phase 4: Summary to User

After generating, display:
1. Brief executive summary (3-5 lines)
2. List of projects covered with activity levels
3. Number of risk alerts
4. Links to saved files (local, Wiki, email status)

## Parameters

Parse arguments from user input:
- `--timerange=Xd` : Override 24h lookback (e.g., `7d` for weekly)
- `--project=NAME` : Filter projects by name regex (e.g., `3M` for 3M projects only)
- `--verbose` : Include full message content
- `--email` : Generate HTML version and send via email
- `--wiki` : Publish to Feishu Wiki
- `--format=html|markdown` : Force output format (default: markdown + HTML for email)

## Quick Reference Commands

| Data | Command |
|------|---------|
| All projects | `basecamp projects list --json --quiet` |
| Project timeline | `basecamp timeline --in <id> --json --quiet` |
| Messages | `basecamp messages list --in <project_id> --json --quiet` |
| Todos | `basecamp recordings list --type todo --in <project_id> --json --quiet` |
| Overdue (cross-project) | `basecamp reports overdue --json --quiet` |
| My assignments | `basecamp assignments list --json --quiet` |
| Search | `basecamp search "query" --json --quiet` |
| Download attachment | `basecamp attachments download <id> --out /tmp/` |
| Download file | `basecamp files download <id> --in <project>` |

## Notes

- CLI handles OAuth token refresh automatically
- Each user sees only the projects they have access to in Basecamp
- For event-driven updates, consider `basecamp webhooks create`
- `basecamp timeline --watch` can monitor real-time activity
