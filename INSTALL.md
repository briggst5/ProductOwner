# Installing the ProductOwner workspace in Cursor

Instructions for setting up [ProductOwner](https://github.com/briggst5/ProductOwner) from scratch.

## What you get

Opening this repo in Cursor automatically loads:

- **26 skills** (`.cursor/skills/`) — PO, RTE, and ceremony workflows
- **6 rules** (`.cursor/rules/`) — FutureState process, ADO/Polarion boundaries
- **6 agents** ([AGENTS.md](AGENTS.md)) — PO Coach, RTE Coordinator, and others
- **Process docs** (`docs/`) — curated summaries, DoR/DoD checklists, templates (in git)
- **Wiki exports** (`reference/`) — synced from Azure DevOps (local only, gitignored)

Live Azure DevOps and Polarion access requires MCP servers and credentials (below).

## Prerequisites

| Requirement | Used for |
|-------------|----------|
| [Cursor](https://cursor.com) (recent version with MCP) | IDE and agent |
| **Node.js 20+** | Azure DevOps MCP (`npx @azure-devops/mcp`) |
| **Python 3.11+** | Polarion MCP, wiki sync script |
| **git** | Clone repositories |
| Network access | FLC-NPD Azure DevOps, Polarion instance |

Access needed from your organization:

- Azure DevOps org **FLC-NPD**, project **FutureState**
- Polarion project (configured in Polarion MCP)
- Personal Access Tokens (PATs) for both, if not using interactive login

## Step 1 — Clone repositories

```bash
git clone https://github.com/briggst5/ProductOwner.git
git clone https://github.com/briggst5/PolarionMCP.git
```

Keep both repos on your machine. You will point `.cursor/mcp.json` at your PolarionMCP clone path.

## Step 2 — Install Polarion MCP

```bash
cd PolarionMCP
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e .
```

Create credentials (never commit this file):

```bash
mkdir -p ~/.config
cp polarion-mcp.env.example ~/.config/polarion-mcp.env
chmod 600 ~/.config/polarion-mcp.env
```

Edit `~/.config/polarion-mcp.env` and set:

- `POLARION_URL`
- `POLARION_USER`
- `POLARION_PAT` or `POLARION_PASSWORD`
- `POLARION_PROJECT`

Smoke test:

```bash
python scripts/test_tools.py list_documents
```

## Step 3 — Configure Azure DevOps MCP credentials

Create `~/.config/azure-devops-mcp.env`:

```bash
mkdir -p ~/.config
chmod 600 ~/.config/azure-devops-mcp.env
```

Example contents:

```bash
ADO_ORG=FLC-NPD
ADO_AUTH=pat
PERSONAL_ACCESS_TOKEN=<base64-encoded :your-ado-pat>
```

Notes:

- `ADO_ORG` is the organization name only (`FLC-NPD`), not the full URL.
- For `ADO_AUTH=pat`, `PERSONAL_ACCESS_TOKEN` must be **base64** of `:YOUR_PAT` (leading colon, empty username):

  ```bash
  echo -n ":YOUR_PAT_HERE" | base64 -w0
  ```

- Alternatives: `ADO_AUTH=interactive` (browser login on first use) or `ADO_AUTH=azcli` (after `az login`).

PAT scopes: work items (read/write as needed), wiki (read), project and team read.

## Step 4 — Update `.cursor/mcp.json` paths

Open `ProductOwner/.cursor/mcp.json` and set the **PolarionMCP path** to your clone:

```json
{
  "mcpServers": {
    "polarion-mcp": {
      "command": "bash",
      "args": [
        "-lc",
        "cd /YOUR/PATH/TO/PolarionMCP && source .venv/bin/activate && polarion-mcp"
      ]
    },
    "azure-devops": {
      "command": "bash",
      "args": [
        "-lc",
        "set -a && source ~/.config/azure-devops-mcp.env && set +a && exec npx -y @azure-devops/mcp \"$ADO_ORG\" -d core wiki work work-items --authentication \"$ADO_AUTH\""
      ]
    }
  }
}
```

Replace `/YOUR/PATH/TO/PolarionMCP` with the absolute path on your machine.

## Step 5 — Open in Cursor

1. **File → Open Folder** and select the `ProductOwner` directory.
2. Cursor should detect `.cursor/mcp.json`.
3. Open **Cursor Settings → MCP** and **Enable** both servers:
   - `polarion-mcp`
   - `azure-devops`
4. If prompted, approve tool access for the agent.

Rules and project skills load from `.cursor/` when this folder is the workspace root. No additional install step is required.

## Step 6 — Verify the setup

Ask the agent in **Agent mode**:

```
List wikis in the FutureState project
```

```
Query my work items in FutureState
```

```
What is our work item hierarchy? Use safe-process-navigator.
```

Expected results:

- ADO returns FutureState wiki or work items
- Polarion responds if credentials are valid
- Process answers cite `docs/futurestate-process.md`, not generic SAFe

### Troubleshooting

| Symptom | Fix |
|---------|-----|
| `CERTIFICATE_VERIFY_FAILED` / SSL errors in WSL (Python, MCP, curl, Chromium) | Run `bash scripts/fix-wsl-corp-certs.sh` (exports Baxter/Zscaler + HRC roots from Windows, updates trust store). Re-run after WSL reinstall. Add `--chromium` for browser NSS. |
| Polarion MCP won't start | Check venv path in `mcp.json`; confirm `polarion-mcp` runs inside the venv |
| ADO auth error | Re-check `~/.config/azure-devops-mcp.env`, PAT scopes, base64 format |
| Wiki tools missing | Confirm `-d core wiki work work-items` in azure-devops args |
| `npx` not found | Install Node.js 20+ |

## Step 7 — Refresh wiki exports (recommended)

Wiki exports under `reference/wiki/` are not in git. After clone, refresh from live Azure DevOps:

```bash
cd ProductOwner
python3 scripts/sync_wiki.py
```

Or in chat:

```
Run wiki-sync
```

For daily auto-refresh:

```
/loop 1d Run wiki-sync — execute python3 scripts/sync_wiki.py and report pages synced and any errors
```

## Step 8 — Use skills and agents

**Skills** — mention by name or intent:

- *"Run backlog-readiness-audit on the platform team backlog"*
- *"Prep PI Planning"*
- *"Rewrite acceptance criteria for story 12345"*

**Agents** — prefix with role (see [AGENTS.md](AGENTS.md)):

- *"Act as PO Coach — break down this feature into stories"*
- *"Act as RTE Coordinator — build this week's ART Sync brief"*
- *"Act as Traceability Auditor — check Polarion links for these stories"*

**Safety:** Rules require confirmation before creating or updating ADO or Polarion work items unless you explicitly ask to apply changes.

## Bootstrap checklist (for setup agents)

If you are a Cursor agent helping a user install this repo:

1. Clone `ProductOwner` and `PolarionMCP`
2. Run `pip install -e .` in the PolarionMCP virtualenv
3. Create `~/.config/polarion-mcp.env` and `~/.config/azure-devops-mcp.env` (never commit)
4. Fix the Polarion path in `ProductOwner/.cursor/mcp.json`
5. Open `ProductOwner` as the Cursor workspace root
6. Enable MCP servers; test ADO wiki list and Polarion query
7. Run `python3 scripts/sync_wiki.py` to populate `reference/wiki/` locally
8. Point the user to [README.md](README.md) for the skill and agent index

## Not included in this repository

- Credentials (`~/.config/*.env`) — per-user, local only
- [PolarionMCP](https://github.com/briggst5/PolarionMCP) — separate clone required
- User-level Cursor MCP in `~/.cursor/mcp.json` — optional; project `.cursor/mcp.json` is sufficient when this folder is the workspace
