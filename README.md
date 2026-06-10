# ProductOwner

Skills, agents, and rules to support Product Owners, RTEs, and agile team members on the FLC FutureState program.

**New here?** See **[INSTALL.md](INSTALL.md)** for setup (clone, MCP credentials, Cursor configuration).

## Process reference

Agile process is documented in the [FutureState Platform.wiki](https://dev.azure.com/FLC-NPD/FutureState/_wiki/wikis/Platform.wiki).

| Resource | Purpose |
|----------|---------|
| [reference/futurestate-process.md](reference/futurestate-process.md) | Consolidated process summary for agents |
| [reference/dor-dod-checklists.md](reference/dor-dod-checklists.md) | DoR/DoD quick-reference gate cards |
| [reference/wiki-index.md](reference/wiki-index.md) | Full wiki page index (auto-generated) |
| [reference/wiki/](reference/wiki/) | All 61 Platform.wiki page exports |
| [reference/templates/](reference/templates/) | Work item & ceremony templates |

## Agents

See **[AGENTS.md](AGENTS.md)** for six specialized personas (PO Coach, RTE Coordinator, Traceability Auditor, Readiness Gatekeeper, Metrics Analyst, Stakeholder Brief Writer).

## Rules (`.cursor/rules/`)

| Rule | Scope |
|------|-------|
| `futurestate-process` | Always — hierarchy, timeboxes, tags, tool routing |
| `ado-polarion-boundary` | Always — ADO vs Polarion |
| `traceability-standards` | Always — requirement links |
| `communication-tone` | Always — status reports |
| `no-unapproved-status-changes` | Always — confirm before writes |
| `work-item-writing-standards` | Templates & draft work items |

## Skills (`.cursor/skills/`)

### Foundation

| Skill | Use when |
|-------|----------|
| `safe-process-navigator` | Process questions, DoR/DoD, tool routing |
| `ado-work-item-steward` | ADO backlog, work items, capacity, iterations |
| `polarion-requirements-steward` | Requirements, SRS, reviews, traceability |
| `wiki-sync` | Refresh wiki exports from Azure DevOps |

### Product Owner

| Skill | Use when |
|-------|----------|
| `pi-objectives-writer` | Draft PI Objectives |
| `feature-refinement` | Groom program Features |
| `story-breakdown` | Feature → Stories |
| `backlog-readiness-audit` | DoR gate / refinement prep |
| `wsjf-prioritization` | Rank Features |
| `acceptance-criteria-coach` | Rewrite AC (Given/When/Then) |

### Release Train Engineer

| Skill | Use when |
|-------|----------|
| `pi-planning-prep` | 2–4 weeks before PI Planning |
| `dependency-tracker` | Cross-team dependencies |
| `iteration-health-check` | Mid/end iteration metrics |
| `scrum-of-scrums-brief` | ART Sync agenda |
| `pi-confidence-calculator` | Confidence vote summary |
| `capacity-planner` | Capacity vs load |

### Team member

| Skill | Use when |
|-------|----------|
| `sprint-planning-assist` | Sprint planning |
| `daily-standup-prep` | Stand-up brief |
| `impediment-logger` | Log blockers / Waiting |
| `definition-of-done-check` | Pre-close validation |
| `release-notes-draft` | Release notes |

### Ceremonies

| Skill | Use when |
|-------|----------|
| `pi-planning-facilitator` | PI Planning materials |
| `backlog-refinement-facilitator` | Refinement pre-read |
| `inspect-adapt-synthesis` | I&A workshop |
| `solution-demo-prep` | System Demo script |
| `architectural-runway-review` | Enabler vs feature balance |

## MCP integrations

Configured in `.cursor/mcp.json`:

| Server | Purpose |
|--------|---------|
| `polarion-mcp` | Requirements, specs, reviews, traceability |
| `azure-devops` | Backlogs, work items, iterations, capacity, wiki |

Azure DevOps domains: `core`, `wiki`, `work`, `work-items`.

Credentials: `~/.config/azure-devops-mcp.env`, `~/.config/polarion-mcp.env`.

## Wiki sync

Refresh all Platform.wiki pages into `reference/wiki/`:

```bash
python3 scripts/sync_wiki.py
```

Or ask the agent: *"Run wiki-sync"*

### Scheduled refresh (`/loop`)

Daily wiki refresh in Cursor chat:

```
/loop 1d Run wiki-sync — execute python3 scripts/sync_wiki.py and report pages synced and any errors
```

Weekly alternative:

```
/loop 7d Run wiki-sync — execute python3 scripts/sync_wiki.py and report summary
```

Background shell helper (optional):

```bash
bash scripts/wiki-sync-loop.sh 86400   # 1 day in seconds
```

## Example prompts

- "Act as PO Coach — break down Feature 12345 into stories"
- "Run backlog-readiness-audit on the Denali team backlog"
- "Prep ART Sync for this week" (RTE Coordinator / `scrum-of-scrums-brief`)
- "Audit traceability for Polarion requirements in the Connex SRS"
- "Is this story ready for sprint planning?" (Readiness Gatekeeper)
- "Run wiki-sync to refresh process docs"
