# msrc-icm-report

An [Agency](https://aka.ms/agency) plugin that generates a recurring report of
**active MSRC ICMs** owned by:

1. **Container Platform / Container Networking**
2. **Windows Server Container Networking / Triage**

The report is ordered by ICM created date and emailed as an HTML table via
WorkIQ (Outlook).

## What it ships

| Component | Path | Purpose |
|---|---|---|
| Custom agent | `agents/msrc-icm-report.md` | The report generator. Edit its **Configuration** section to change teams, recipient, subject, or the SLA table. |

**MCP servers used** (all built-in to Agency, launched via the agent's
`mcp-servers` block): `icm` (incident data), `ado` / `bluebird` (task groups &
PRs), `workiq` (Teams chat lookup + email send).

## Report columns

| # | Column | Source / derivation |
|---|---|---|
| 1 | MSRC Number | MSRC case number — **hyperlinked to the ADO task group** |
| 2 | Created Date | ICM created date |
| 3 | Resolve By | R-ICM MSRC tab / ADO task group (fallback: Created + SLA) |
| 4 | Severity | MSRC severity — R-ICM MSRC tab / ADO task group |
| 5 | ICM Severity | Sev 1–4 from the incident |
| 6 | Assignee | Current owner/assignee |
| 7 | ICM Name | Incident title |
| 8 | Feature team ICM | ICM owned by the two teams — **hyperlinked** to IcM |
| 9 | WSD ICM | WSD primary Release ICM — **hyperlinked** to IcM |
| 10 | PR & Status | Active/Completed PR from ADO (via child bugs) — **hyperlinked**, else `No PR` |
| 11 | Teams Chat | Per-MSRC Teams group chat from WorkIQ — **hyperlinked**, else `No chat` |

Any MSRC that is **overdue or due within 14 days** has its **entire row marked
red** and is listed in an overdue/nearing-due callout above the table.

### Resolve-By SLA (fallback when MSRC tab not assessed)

Counted from the ICM **creation date**:

| MSRC severity | Days from Created |
|---|---|
| Critical | 45 |
| Important | 90 |
| Moderate | 180 |
| Low | 180 |

## Install & run

Once published to the Agency marketplace:

```bash
# From the playground marketplace
agency copilot --plugin market:msrc-icm-report@playground --agent msrc-icm-report:msrc-icm-report
```

Iterate locally straight from disk (never cached):

```bash
agency plugin check ./msrc-icm-report
agency copilot --plugin local:./msrc-icm-report --agent msrc-icm-report:msrc-icm-report
```

The `icm` and `workiq` MCP servers authenticate via your interactive Entra
login.

> **Note:** The Teams Chat column populates only for MSRCs whose group chat the
> **running identity is a member of** (WorkIQ sees the signed-in user's chats).
> Fine for a personal run; a cloud schedule must run under a member identity.

## Authors

Prince Pereira (`ppereira@microsoft.com`)
