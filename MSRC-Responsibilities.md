# MSRC Case Responsibilities: PET (Partner Engineering Team) vs WSD (Windows Servicing Delivery)


## Initial Steps (All Severities)

1. When an MSRC case is reported, the MSRC team creates **two R-ICMs**: one assigned to **WSD** (primary) and one to **PET** (secondary).
2. The assigned WSD Dev creates a **Teams group chat** and adds WSD, PET, and V&M Engineer members.
3. WSD and PET jointly assess the severity. If the AI-assessed severity in the Task Group differs from the team's assessment, fill out [aka.ms/MSRCCaseUnblock](https://aka.ms/MSRCCaseUnblock) and/or inform the MSRC engineer about any roadblocks.

### How to Find the Assigned Release

To determine which release (e.g., 2026.09B, 2026.10B) an MSRC case is targeting, use either method:

| Method | Navigation Path |
|---|---|
| **From the MSRC Task Group** | Task Group → Main → Release |
| **From the R-ICM** | R-ICM → MSRC Tab → "Resolve By" date → the **B-release of that month** |

---


## Branch Types

| Term | What It Is | Examples |
|---|---|---|
| **Active Branch (AB)** | The primary development branch where the fix is initially authored and reviewed. This is where the code change originates before flowing to other branches. | `GE_CURRENT`, `rs_wsd_cfe_dev` |
| **Servicing Branches** | Release-specific branches that receive security updates for currently supported Windows versions. Fixes are either ported manually by WSD or flow automatically via the RI pipeline from `GE_CURRENT`. | `ge_release_svc_xxB`, `br_release_svc_xxB` |
| **Other Servicing / Legacy Branches** | Branches for older Windows versions still under extended support. These always require manual porting by WSD and only apply to Important/Critical cases. | `Lt_release_svc_xxB` `fe_release_svc_xxB`, `ni_release_svc_xxB`, `vb_release_svc_xxB`, `rs5_release_svc_xxB`, `rs1_release_1`, `winblue_ltsb`, `win8_ldr`, Hotpatching branches |

---


## MSRC SLAs

| Severity | SLA |
|---|---|
| Critical | 45 days |
| Important | 90 days |
| Moderate | 180 days |
| Low | 180 days |

> **Note:** SLA days are counted from the **case creation date in ICM**. If the severity is changed after initial assessment, the assigned WSD Dev must ensure the updated severity is reflected in both **ICM** and the corresponding **ADO bugs**.

### What Happens if Assessment Is Delayed?

Assessment must be completed as soon as the ICM is created — **especially for Critical cases**, where the 45-day SLA clock is already running. The assigned WSD Dev will receive follow-up emails and Teams messages from upper management / MSRC team if the assessment is not concluded and the fix is not shipped within the SLA. For Critical cases, escalations happen quickly. Do not wait — treat assessment completion as a **day-one priority**.

---


## MSRC Timeline Checkpoints

| Milestone | Timeframe | Action Required |
|---|---|---|
| ICM created | Day 0 - 10 | WSD creates Teams chat, starts severity assessment with PET. |
| Severity confirmed | ASAP (Critical = day-one priority) | Finalize assessment. Fill out [aka.ms/MSRCCaseUnblock](https://aka.ms/MSRCCaseUnblock) if severity differs from AI assessment. |
| Action Plan submitted | ~10 business days after severity confirmed | Fill out template in primary ICM. MSRC monitors this. |
| Fix finalized and reviewed | Well before check-in deadline | PET + WSD collaborate on fix; PET reviews for AB. |
| Check-in deadline | 1st Friday, one month before release | Fix must be checked in. Check [aka.ms/bugben](https://aka.ms/bugben). |
| Release | Patch Tuesday | Fix ships to customers. |

---


## Containment

**Containment is required for all branches and all severities.** Every MSRC fix must ship behind a feature flag (KIR) to enable controlled rollout and rollback.

| Severity | Containment Style | Flag Example |
|---|---|---|
| **Important / Critical** | Security-style KIR flag | `Feature_MSRCXXXX` |
| **Moderate / Low** | Non-security CD-style feature flag | See [CD guidance on OSGWiki](https://www.osgwiki.com/wiki/Delivering_Windows_via_Continuous_Delivery#Moderate_and_Low_Windows_Security_Bug_Guidance_and_CD) for naming examples |

---


## Important / Critical Severity


| Activity | Owner |
|---|---|
| Develop and finalize the fix | PET + WSD |
| Code review and sign-off for Active Branch (AB) | PET + WSD |
| Port fix to all servicing and legacy branches + PR sign-off | **WSD** |
| Check-in by deadline | **WSD** |
| Containment: ship with KIR flag (`Feature_MSRCXXXX`) | PET + WSD |

> **Check-in deadlines** are **one month prior** to the release, typically the **1st Friday** of that month.
> Example: a fix targeting **2026.10B** has a check-in deadline of **September 4th**.
> Check [aka.ms/bugben](https://aka.ms/bugben) for exact deadlines per release.

### AB Bug Check-in: Which Branch to Use?

The fix must reach the **RS (MAIN)** branch by the release date. There are two branch paths, and **the choice is based on timing, not ownership** — whoever owns the AB bug can use either branch.

> **Simple rule:** If the fix is ready **≥ 2.5 weeks before** the standard check-in deadline (per [aka.ms/bugben](https://aka.ms/bugben)), use **`GE_CURRENT`**. Otherwise, use **`rs_wsd_cfe_dev`** to meet the deadline.

For full CodeFlow details, see: [Windows Continuous Delivery CodeFlow Map](https://www.osgwiki.com/wiki/Windows_Continuous_Delivery_CodeFlow_Map#GE_to_BR_CD_Timeline_(Post_BR_Final_build):)

#### `GE_CURRENT` — Use when there is enough lead time

| | |
|---|---|
| **How it works** | Code checked into `GE_CURRENT` automatically RIs to MAIN, and also flows to GE and BR servicing branches. |
| **Upside** | One check-in covers AB, GE, and BR — no separate porting needed for those branches. |
| **Deadline** | **~2.5 weeks earlier** than the standard BigBen deadline, because the RI pipeline needs time to propagate the change to MAIN. |
| **Use when** | The fix is ready well ahead of the standard deadline, giving enough buffer for the RI pipeline. |

#### `rs_wsd_cfe_dev` — Use when the deadline is tight

| | |
|---|---|
| **How it works** | Code goes directly to MAIN, using the standard check-in deadlines per [aka.ms/bugben](https://aka.ms/bugben). |
| **Upside** | Guarantees the fix reaches MAIN on time. Uses the standard BigBen deadline. |
| **Downside** | WSD must **separately port** the fix to GE and BR servicing branches. |
| **Use when** | The fix is not ready 2.5 weeks early, or the `GE_CURRENT` RI deadline has already passed. |

> **Note:** If the AB bug is **assigned to PET**, PET owns end-to-end delivery including ADO bug updates. However, for Important/Critical cases, WSD should evaluate which branch path meets the release timeline and take action accordingly.

### Branches and Ownership (Important / Critical)

| Branch | Owner | Notes |
|---|---|---|
| **AB** (`GE_CURRENT` or `rs_wsd_cfe_dev`) | PET + WSD | See branch options above. |
| **GE + BR servicing** (`ge_release_svc_xx`, `br_release_svc_xx`) | **WSD** ports | Fixes checked into `GE_CURRENT` flow to GE and BR automatically via RI. Fixes checked into `rs_wsd_cfe_dev` require WSD to port to both GE and BR. |
| **LT servicing** (`lt_release_svc_xx`) | **WSD** ports | Always manually ported by WSD. |
| **Legacy** (FE, NI, VB, RS5, RS1, WinBlue, Win8, Hotpatching) | **WSD** ports | Always manually ported by WSD. |

---


## Moderate / Low Severity

These cases follow the **Continuous Delivery (CD)** process. They are treated as **non-security work items** and ship within the **180-day SLA**.

### Bug Assignment & Ownership

- WSD assigns the **MSRC Task Group** and the **AB, GE, BR bugs** to **PET**.
- All other bugs (legacy branches: FE, NI, VB, RS5, RS1, WinBlue, Win8, Hotpatching) are closed as **"Won't Fix"**.
- **Exception (Moderate only):** The **LT bug** stays with **WSD** — WSD will backport and check in the fix for LT.

### Responsibilities

| Activity | Owner |
|---|---|
| Parent MSRC Task Group to an approved CD scenario or monthly bundle | PET |
| Fix development, code review, and check-in for AB/GE/BR | PET |
| Containment: ship with non-security CD-style feature flag | PET |
| LT bug backport and check-in (Moderate only) | **WSD** |

### How It Ships

- PET checks the fix into **`GE_CURRENT`**. The CD pipeline automatically RIs the change to **MAIN**, and it also flows to **GE** and **BR** servicing branches — no separate porting is needed.
- **GE, BR, and RS (MAIN)** ship the fix **EnabledByDefault**, as per the CD guidance.
- For full guidance, see: [Delivering Windows via Continuous Delivery – Moderate/Low Guidance](https://www.osgwiki.com/wiki/Delivering_Windows_via_Continuous_Delivery#Moderate_and_Low_Windows_Security_Bug_Guidance_and_CD)

---


## Branches by Severity (Complete View)

| Severity | RS (MAIN) | GE Servicing | BR Servicing | LT Servicing | Legacy (FE, NI, VB, RS5, RS1, WinBlue, Win8, Hotpatching) |
|---|---|---|---|---|---|
| **Critical** | ✅ PET + WSD | ✅ WSD ports | ✅ WSD ports | ✅ WSD ports | ✅ WSD ports |
| **Important** | ✅ PET + WSD | ✅ WSD ports | ✅ WSD ports | ✅ WSD ports | ✅ WSD ports |
| **Moderate** | ✅ PET (via RI) | ✅ PET (auto via `GE_CURRENT`) | ✅ PET (auto via `GE_CURRENT`) | ✅ WSD backports | ❌ Won't Fix |
| **Low** | ✅ PET (via RI) | ✅ PET (auto via `GE_CURRENT`) | ✅ PET (auto via `GE_CURRENT`) | ❌ Won't Fix | ❌ Won't Fix |

> **Containment is required for all branches and severities.** Imp/Crit uses `Feature_MSRCXXXX` KIR flags; Mod/Low uses non-security CD-style feature flags.

---


## Missed Deadlines & Escalations

| Scenario | What Happens |
|---|---|
| **PET misses `GE_CURRENT` check-in deadline** | WSD checks in the fix via `rs_wsd_cfe_dev` as a fallback to ensure it reaches MAIN on time. |
| **RS (MAIN) deadline missed entirely** | Check-in after the deadline requires **VP approval** from the WSD side. The fix may slip to the next release if approval is not obtained. |
| **Action Plan not submitted (~10 business days)** | Once severity is confirmed, the engineering team must fill out the **Action Plan** template posted in the primary ICM within ~10 business days. MSRC runs automated checks and will follow up if this is overdue. |
| **Fix not shipped within SLA** | MSRC monitors timelines continuously with automated checks. The assigned WSD Dev will receive escalation emails and Teams messages. For **Critical cases**, escalations happen rapidly. |

---


## Quick Reference

| | Critical | Important | Moderate | Low |
|---|---|---|---|---|
| **SLA** | 45 days | 90 days | 180 days | 180 days |
| **Fix development** | PET + WSD | PET + WSD | PET | PET |
| **AB check-in** | PET or WSD ([see branch options](#ab-bug-check-in-which-branch-to-use)) | PET or WSD ([see branch options](#ab-bug-check-in-which-branch-to-use)) | PET | PET |
| **GE/BR porting** | WSD (or auto if via `GE_CURRENT`) | WSD (or auto if via `GE_CURRENT`) | Auto (CD pipeline) | Auto (CD pipeline) |
| **LT porting** | WSD | WSD | WSD | N/A |
| **Legacy branches** | WSD | WSD | Won't Fix | Won't Fix |
| **Containment** | KIR flag (`Feature_MSRCXXXX`) | KIR flag (`Feature_MSRCXXXX`) | CD feature flag | CD feature flag |
| **Shipping model** | Monthly B-release | Monthly B-release | CD (EnabledByDefault) | CD (EnabledByDefault) |
