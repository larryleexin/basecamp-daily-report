# Basecamp Daily Report Plugin

Basecamp 项目日报生成器 — 自动拉取项目数据，AI 分析生成中英文双语摘要报告。

**A Claude Code / Cowork plugin** that pulls Basecamp project data via the official CLI and generates AI-analyzed daily summaries with unified formatting.

## Features

- **One command**: `/basecamp-summary` generates a complete daily report
- **Bilingual**: English primary, Chinese secondary (configurable)
- **Unified format**: Consistent HTML email template across all users
- **Multi-output**: Local Markdown + Feishu Wiki + HTML Email
- **Per-user projects**: Each user sees only their accessible Basecamp projects

## Quick Start

### 1. Install the Plugin

**Claude Code CLI:**
```bash
# From GitHub
/plugin marketplace add genimex/basecamp-daily-report

# Or load locally for testing
claude --plugin-dir /path/to/basecamp-daily-report
```

**Cowork (Claude Desktop):**
- Open Claude Desktop → Cowork tab → Plugin browser → Search "basecamp-daily-report" → Install

### 2. Set Up Basecamp CLI (one-time)

```bash
# Install Basecamp CLI
curl -fsSL https://get.basecamp.sh | sh

# Or if already installed, check version
basecamp --version

# Authenticate with your Basecamp account
basecamp auth login
# → Opens browser for OAuth authorization
# → Select your Basecamp account (e.g., Genimex 3609478)

# Verify
basecamp auth status
basecamp projects list
```

### 3. Use It

```bash
# Basic daily summary (all accessible projects)
/basecamp-summary

# 3M projects only
/basecamp-summary --project=3M

# Weekly summary
/basecamp-summary --timerange=7d

# Generate + send email + publish to Wiki
/basecamp-summary --email --wiki

# Full options
/basecamp-summary --project=3M --timerange=7d --email --wiki --verbose
```

## Configuration

Plugin configuration via `userConfig` (set during install or update later):

| Config Key | Description | Example |
|---|---|---|
| `basecamp_account_id` | Your Basecamp account ID | `3609478` |
| `email_recipients` | Comma-separated email list | `larry@example.com,team@example.com` |
| `feishu_wiki_node` | Feishu Wiki parent node token | `BQr6wO0ZGiLm2ikOdN1cUCPWnOd` |
| `report_language` | Language mode | `bilingual` (default), `en`, `cn` |

## Report Format

All reports follow the standard HTML template (`templates/report-email.html`):

1. **Title** — Blue header with date
2. **Executive Summary** — Key highlights with bold numbers
3. **Project Updates** — Per-project hot topics, organized by recency
4. **Risk Alerts** — Color-coded table (red=high, orange=medium)
5. **Key Insights** — Cross-project observations
6. **Footer** — Generator info + Wiki link

## Prerequisites

| Dependency | Required? | Install |
|---|---|---|
| Basecamp CLI | **Yes** | `curl -fsSL https://get.basecamp.sh \| sh` |
| Basecamp OAuth | **Yes** | `basecamp auth login` |
| lark-cli | Optional (for Wiki) | [lark-cli docs](https://github.com/nicognaW/lark-cli) |
| Composio MCP | Optional (for email) | Configured via Claude Code MCP settings |

## Scheduling (Optional)

Set up automatic daily reports:

```
# In Claude Code, create a scheduled task:
# Weekdays at 8:00 AM local time
Task ID: basecamp-daily-report
Cron: 0 8 * * 1-5
Prompt: /basecamp-summary --email --wiki
```

## License

MIT
