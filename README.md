# MSRC ICM Timeline Report

An [Agency](https://aka.ms/agency) custom agent that generates a recurring
report of **active MSRC ICMs** owned by:

1. **Container Platform / Container Networking**
2. **Windows Server Container Networking / Triage**

The report is ordered by ICM created date and emailed as an HTML table to
`ppereira@microsoft.com`.

## Report columns

| # | Column | Source / derivation |
|---|---|---|
| 1 | MSRC Number | MSRC case number (R-ICM MSRC tab, or feature-ICM summary) |
| 2 | ICM Name | Incident title |
| 3 | Feature team ICM | The ICM owned by the two teams — **hyperlinked** to IcM |
| 4 | WSD ICM (R-ICM) | WSD primary Release ICM — **hyperlinked** to IcM |
| 5 | ADO Task Group | Found via **ADO/Bluebird** by MSRC number — **hyperlinked** |
| 6 | PR | Active/Merged PR from **ADO** — **hyperlinked** + status, else `No PR` |
| 7 | Assignee | Current owner/assignee |
| 8 | ICM Severity | Sev 1–4 from the incident |
| 9 | MSRC Severity | **R-ICM MSRC tab** (fallback: ADO task group) |
| 10 | Resolve By | **R-ICM MSRC tab** (fallback: Created date + SLA) |

An **at-risk** callout is included above the table: any MSRC whose Resolve By is
within 14 days (or overdue) **and** has no merged PR is highlighted and listed.

Data source = built-in `icm` MCP; ADO task group & PRs = `ado`/`bluebird` MCP;
email = built-in `workiq` MCP.

### Resolve-By SLA (fallback when MSRC tab not assessed)

Per `MSRC-Responsibilities.md`, counted from the ICM **creation date**:

| MSRC severity | Days from Created |
|---|---|
| Critical | 45 |
| Important | 90 |
| Moderate | 180 |
| Low | 180 |

## Files

- `.github/agents/msrc-icm-report.agent.md` — the custom agent (data source =
  built-in `icm` MCP, email = built-in `workiq` MCP). Edit the **Configuration**
  section to change teams, recipient, subject, or the SLA table.
- `.azuredevops/policies/agency-schedule.yml` — cloud cron schedule (every 2 days).

## Run it manually (local)

```powershell
agency copilot --agent msrc-icm-report
```

Both `icm` and `workiq` MCP servers authenticate via your interactive Entra
login. The current folder is already a trusted Agency folder.

## Run it every 2 days WITHOUT keeping your laptop on

Agency agents do not self-schedule. Pick one of these **cloud-hosted** options so
nothing depends on your machine being powered on:

### Option A — Agency Schedules (recommended, simplest)

Commit `.azuredevops/policies/agency-schedule.yml` to your repo's **default
branch**. The Agency service fires the agent on the cron cadence in the cloud.

- Requires the repo to be enrolled in the **Agency schedules private preview**
  (currently closed to new repos — request access / check status with the Agency
  team: https://aka.ms/agency/engage).
- No laptop, no VM, no always-on machine required.

### Option B — Scheduled Azure DevOps pipeline (generally available)

Use a scheduled ADO pipeline that extends the **1ES Agency Pipeline Template** and
runs the agent headlessly. MCP auth in pipelines uses **Workload Identity
Federation (WIF)** / OBO — no interactive login. Both `icm` and `workiq` are
supported in pipelines.

Sketch (`azure-pipelines.yml`):

```yaml
schedules:
  - cron: "0 6 */2 * *"        # every 2 days at 06:00 UTC
    displayName: MSRC ICM report
    branches: { include: [ main ] }
    always: true

extends:
  template: v1/1ES.Agency.PipelineTemplate.yml@1esPipelines
  parameters:
    # Pair each MCP name with a WIF-enabled service connection pre-authorized
    # for that MCP's resource (see the Agency "MCP Auth in Pipelines" doc).
    mcpConfiguration:
      icm:
        serviceConnection: "[App] ICM MCP Auth"
      workiq:
        serviceConnection: "[App] WorkIQ MCP Auth"
    # ...template steps run: agency copilot --agent msrc-icm-report
```

See the Agency docs: *Azure DevOps → MCP Auth in Pipelines* and
*Agency Schedules* for full setup, and confirm the `workiq` service connection /
mailbox identity is authorized to send mail non-interactively.

> Note: emailing via WorkIQ from a headless pipeline depends on the initiating /
> app identity having Mail.Send. Validate a first run before trusting the cadence.

### Option C — Any always-available scheduler you already own

Azure Automation runbook, an Azure Function timer trigger, or a small always-on
VM can invoke `agency copilot --agent msrc-icm-report` on a timer. These avoid
your laptop but you manage the compute and its Entra auth yourself.

## Recommended answer to "no laptop resources / not always-on"

Use **Option A** if your repo can join the schedules preview; otherwise use
**Option B** (scheduled ADO pipeline with WIF). Both run entirely in Microsoft
cloud infrastructure, so your laptop can be off.
