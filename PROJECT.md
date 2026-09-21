<!--
  PROJECT.md — living reference doc for the MSRC ICM Timeline Report.
  Keep this current: update Status, Milestones, Risks, and the Changelog each cycle.
  Placeholders are marked <like-this> — replace them as details firm up.
-->

# MSRC ICM Timeline Report — Project Details

> A living reference document. Update the **Status**, **Milestones / Roadmap**,
> **Risks**, and **Changelog** sections whenever things change.

| | |
|---|---|
| **Project name** | MSRC ICM Timeline Report |
| **Type** | Agency custom-agent plugin |
| **Repository** | `MSRCTimelines` |
| **Doc last updated** | 2026-09-22 |
| **Current version** | 1.0.0 |
| **Lifecycle stage** | <Prototype / Local-only / Playground / Curated> |

---

## 1. Overview / Purpose

An [Agency](https://aka.ms/agency) plugin that generates a recurring report of
**active MSRC ICMs** owned by the two teams in scope, orders them by created
date, flags overdue / near-due cases, and emails a self-contained HTML table via
WorkIQ (Outlook).

**Problem it solves:** MSRC incident status is scattered across IcM, WSD R-ICMs,
ADO task groups & child-bug PRs, and Teams chats. Assembling a current report by
hand is slow, inconsistent, and easy to let Resolve-By deadlines slip.

**Value:** one autonomous, on-demand or scheduled report that consolidates every
source and highlights what needs attention.

---

## 2. Owner & Contacts

| Role | Name | Contact |
|---|---|---|
| Owner / author | Prince Pereira | `ppereira@microsoft.com` |
| Report recipient | Prince Pereira | `ppereira@microsoft.com` |
| Backup / secondary | <name> | <email> |
| Team | <team name / alias> | <alias@microsoft.com> |

---

## 3. Current Status

- **State:** <e.g. Working locally; validated with `agency plugin check`>
- **Last successful run:** <date / N/A>
- **Deployed schedule:** <None / Agency Schedules / ADO pipeline / other>
- **Blocking items:** <none / describe>
- **Notes:** <free text>

---

## 4. Scope

**Owning teams (report scope):**
1. `Container Platform` / `Container Networking`
2. `Windows Server Container Networking` / `Triage`

**Incident filter:** Active (open) incidents only, sorted by ICM Created date
(oldest first).

**Data sources (MCP servers):**

| Source | MCP | Used for |
|---|---|---|
| IcM | `icm` | Active ICMs, WSD R-ICM, MSRC-tab fields |
| ADO | `ado` / `bluebird` | Task groups, child bugs, PR status |
| Teams / Outlook | `workiq` | Per-MSRC Teams chat lookup + email send |

**ADO org / project:** `microsoft` / `OS`.

---

## 5. Architecture / Components

```
MSRCTimelines/
├── msrc-icm-report/                 # the publishable Agency plugin
│   ├── plugin.json                  # Copilot-native manifest (name, version, author)
│   ├── agency.json                  # engines=["copilot"], category, platforms
│   ├── README.md                    # plugin usage docs
│   └── agents/
│       └── msrc-icm-report.md       # the custom agent (logic + mcp-servers)
├── .github/agents/msrc-icm-report.agent.md   # original standalone agent
├── .azuredevops/policies/agency-schedule.yml # cloud cron schedule
├── MSRC-Responsibilities.md         # MSRC process source-of-truth (SLAs, R-ICM, etc.)
├── MSRC-ICM-Report.pptx             # overview slide deck
├── PROJECT.md                       # this file
└── README.md                        # repo overview
```

**Report pipeline (per run):**
1. Query active ICMs for both teams (`icm`).
2. Read WSD R-ICM / ADO task group for MSRC severity & Resolve-By.
3. Traverse child bugs for linked PRs (`ado`/`bluebird`).
4. Match per-MSRC Teams chat (`workiq`).
5. Compute RED flag (overdue or ≤14 days) and sort by Created date.
6. Build inline-styled HTML table → email via `workiq` → print run summary.

**Report columns:** MSRC Number (→ADO), Created Date, Resolve By, MSRC Severity,
ICM Severity, Assignee, ICM Name, Feature team ICM (→IcM), WSD ICM (→IcM),
PR & Status (→ADO), Teams Chat (→Teams).

**Resolve-By SLA fallback** (from ICM Created date, when MSRC tab unassessed):
Critical 45d · Important 90d · Moderate 180d · Low 180d.

---

## 6. How to Run

**Local, manual:**
```bash
agency plugin check ./msrc-icm-report
agency copilot --plugin local:./msrc-icm-report --agent msrc-icm-report:msrc-icm-report
```
At the prompt: `Generate and email the active MSRC ICM report now.`

Both `icm` and `workiq` authenticate via interactive Entra login on first use.

---

## 7. Schedule / Cadence

- **Target cadence:** every 2 days (see `.azuredevops/policies/agency-schedule.yml`).
- **Hosting options (no laptop required):**
  - **A — Agency Schedules** (private preview): commit the schedule YAML to the default branch.
  - **B — Scheduled ADO pipeline** extending the 1ES Agency template; MCP auth via Workload Identity Federation (WIF).
  - **C — Any always-on scheduler** (Azure Automation, Function timer, VM).
- **Current choice:** <A / B / C / none>
- ⚠️ Teams Chat + email require a **member identity** (WorkIQ only sees the running identity's chats/mailbox).

---

## 8. Milestones / Roadmap

| Milestone | Status | Target date | Notes |
|---|---|---|---|
| Agent authored & running locally | ✅ Done | 2026-09 | |
| Packaged as Agency plugin | ✅ Done | 2026-09 | `msrc-icm-report/` scaffolded, `plugin check` passes |
| First scheduled cloud run | ⬜ Todo | <date> | pick Option A/B/C |
| Published to playground marketplace | ⬜ Todo | <date> | PR to agency-microsoft/playground |
| Promoted to curated marketplace | ⬜ Todo | <date> | submission issue flow |
| <next milestone> | ⬜ | <date> | |

---

## 9. Risks / Open Issues

| # | Risk / issue | Impact | Mitigation | Status |
|---|---|---|---|---|
| 1 | WorkIQ sees only the running identity's Teams chats | Teams Chat column blank under a non-member identity | Run schedule under a member identity | Open |
| 2 | Headless email needs `Mail.Send` on the app/initiating identity | Report not delivered | Validate a first pipeline run before trusting cadence | Open |
| 3 | R-ICM MSRC tab may be ACL-denied | Missing severity / Resolve-By | Fall back to ADO task group; else SLA compute | Mitigated |
| 4 | Agency Schedules preview closed to new repos | Option A unavailable | Use Option B (ADO pipeline + WIF) | Open |
| 5 | <new risk> | <impact> | <mitigation> | <open/closed> |

---

## 10. Changelog / Update Log

| Date | Version | Change | By |
|---|---|---|---|
| 2026-09-22 | 1.0.0 | Packaged agent as Agency plugin (`msrc-icm-report/`); added PROJECT.md and overview deck | Prince Pereira |
| <date> | <ver> | <change> | <who> |

---

## 11. Links & References

- Agency: https://aka.ms/agency
- Agency Marketplace: https://aka.ms/agency/marketplace
- Playground repo: https://github.com/agency-microsoft/playground
- Curated submission form: https://aka.ms/agency/publish-plugin
- Agency engagement / preview requests: https://aka.ms/agency/engage
- MSRC process source-of-truth: `MSRC-Responsibilities.md`
- Plugin usage docs: `msrc-icm-report/README.md`
- Overview deck: `MSRC-ICM-Report.pptx`
