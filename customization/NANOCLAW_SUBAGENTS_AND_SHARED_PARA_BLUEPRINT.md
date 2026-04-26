# NanoClaw Customization Blueprint: Subagents + Shared PARA Knowledge

## Purpose
This document defines a concrete customization target for this NanoClaw fork:
1. Which subagents to create.
2. How the main orchestrator agent should delegate.
3. Which shared knowledge structure all agents use (PARA).
4. Which additional setup tasks should be implemented (Signal, GitHub auth, backup to repository).

This blueprint is based on:
- The role model in the local Family-assistant repository (Executive + Support agents).
- The isolation/orchestration model in NanoClaw (agent groups, channel wiring, per-group workspace, scoped memory).

---

## Design Principles

### 1) Keep isolation, standardize contracts
NanoClaw should keep per-agent isolation (separate agent groups/workspaces), but all agents must speak the same data contract and directory contract for shared knowledge.

### 2) Executive-first delegation
Primary user interaction should happen through Executive agents. Support agents are specialists called by Executive agents.

### 3) PARA as the universal memory map
Every agent can read the same PARA hierarchy. Write permissions are role-specific to avoid conflicts.

### 4) Deterministic operations
Recurring work (backups, checks, summaries) should be scheduler-driven with explicit logs and acceptance criteria.

---

## Target Agent Topology

## Main Orchestrator
- Current group in local fork: family
- Recommended orchestrator display name: Family Control Tower
- Role:
  - Receives cross-domain requests.
  - Routes to Executive or Support agents.
  - Enforces standards (templates, naming, backup policy, priority rules).
  - Publishes weekly system health report.
- Permissions:
  - Read all shared PARA folders.
  - Write to system coordination files and reports.

## Executive Subagents (Primary)
Current local agent set (with actual assistant names):
1. Executive-Suse (assistant name: Quatschkopf)
2. Executive-Andy (assistant name: Pauline)
3. Executive-Felix (assistant name: Fabio)
4. Executive-Marie (assistant name: Clarie)
5. Executive-Rosa (assistant name: Lila)

Executive responsibilities:
- Own personal and role-specific tasks.
- Keep personal TODO list current.
- Escalate cross-family conflicts to the Main Orchestrator.
- Delegate specialist tasks to Support agents.

## Support Subagents (Specialists)
Current local specialist roster:
1. Support-Kalender
2. Support-Putzfrau
3. Support-Babysitter
4. Support-Hausmeister
5. Support-Gaertner
6. Support-Urlaubsplaner
7. Support-Mechaniker
8. Support-Familienberater
9. Support-PersonalAssistant
10. Support-ManagementConsultant
11. Support-Steuerberater
12. Support-Knaufi

Target roster for this fork (required):
1. Support-Kalender
2. Support-Putzfrau
3. Support-Babysitter
4. Support-Hausmeister
5. Support-Gaertner
6. Support-Urlaubsplaner
7. Support-Mechaniker
8. Support-Familienberater
9. Support-PersonalAssistant
10. Support-ManagementConsultant
11. Support-Steuerberater
12. Support-Knaufi
13. Support-Scientist

Local notes from current instructions:
- Support-Knaufi is Felix's dedicated homework and learning coach, mainly delegated by Executive-Felix (Fabio).
- Support-Steuerberater tracks tax-deductible expenses and filing deadlines for Andy and Suse.
- Support-Kalender is the only interface to calendar sync tasks in the family setup.
- Support-Scientist remains required for Suse (research, talks, posters, and PhD supervision) and should be present in this fork even if missing in the current local nanoclaw tree.

Support responsibilities:
- Operate within one domain.
- Produce actionable outputs (plans, checklists, reminders, summaries).
- Write updates into designated PARA targets.
- Return completion status to requesting Executive agent.

## Delegation Routing Matrix (actual usage baseline)
- Executive-Suse (Quatschkopf) -> Putzfrau, Babysitter, Urlaubsplaner, Familienberater, PersonalAssistant, Kalender, Scientist
- Executive-Andy (Pauline) -> Hausmeister, Gaertner, Mechaniker, Urlaubsplaner, ManagementConsultant, Steuerberater, Kalender
- Executive-Felix (Fabio) -> Knaufi (primary), Kalender
- Executive-Marie (Clarie) -> Kalender, Familienberater (when needed)
- Executive-Rosa (Lila) -> Kalender, Putzfrau/Babysitter for logistics as needed

---

## Shared PARA Knowledge Architecture

## Shared root
Use one mounted shared root available to all relevant agents:
- /shared-knowledge

Inside shared root, enforce this structure:
- /shared-knowledge/Projects
- /shared-knowledge/Areas
- /shared-knowledge/Resources
- /shared-knowledge/Archives
- /shared-knowledge/System

### Projects
Purpose: Time-bound outcomes with explicit end state.
- Required metadata in each project note:
  - owner
  - due_date
  - status
  - related_agents
  - last_reviewed
- Required sections:
  - Goal
  - Scope
  - Tasks
  - Risks
  - Decision Log

### Areas
Purpose: Ongoing responsibilities without fixed end.
Recommended subfolders:
- /Areas/Family
- /Areas/House
- /Areas/Garden
- /Areas/Vehicles
- /Areas/Health
- /Areas/Finances
- /Areas/School

Each area should contain:
- Current state
- Standard operating checklist
- Open loops
- KPIs or weekly indicators

### Resources
Purpose: Reusable references.
Recommended subfolders:
- /Resources/Templates
- /Resources/Knowledge
- /Resources/Checklists
- /Resources/Playbooks

Rules:
- No active TODO execution lives here.
- Keep version/date marker for changed procedures.

### Archives
Purpose: Completed or inactive work.
Rules:
- Move completed projects here with completion date.
- Keep postmortem summary when applicable.

### System
Purpose: Governance and cross-agent operations.
Recommended files:
- /System/agent-registry.md
- /System/delegation-protocol.md
- /System/backup-policy.md
- /System/change-log.md
- /System/weekly-system-report.md

---

## Write Access Model

To avoid conflict and accidental overwrite:
- Main Orchestrator:
  - Write access: all System files, cross-agent summaries.
- Executive agents:
  - Write access: own personal notes, assigned projects, shared family planning files.
- Support agents:
  - Write access: their domain output files only.
- Global rule:
  - Any agent can propose edits outside its scope, but must record proposal in a handoff note instead of direct overwrite.

---

## Delegation and Handoff Protocol

When an agent delegates work:
1. Create handoff entry in /System/delegation-log.md.
2. Include:
  - requester
  - receiver
  - objective
  - input files
  - expected output file
  - deadline
3. Receiver writes result and completion status.
4. Requester confirms acceptance or requests revision.

Handoff states:
- queued
- in_progress
- blocked
- delivered
- accepted

---

## Required Subagent Prompt Contracts

Each subagent should include in its system prompt:
- Identity and scope boundaries.
- Allowed write targets.
- Escalation behavior when out of scope.
- Priority order: safety/privacy -> deadlines -> dependencies -> optimization.
- Required output format for task completion.

Minimum completion output format:
- Summary
- Files changed
- Open risks
- Next actions

---

## Upstream Update Policy

- NanoClaw ships a `/update-nanoclaw` skill (`.claude/skills/update-nanoclaw/SKILL.md`) for safe on-demand upstream sync.
- Updates are **not automatic** — they require running `/update-nanoclaw` in Claude Code.
- After deployment: Orchestrator must have a **weekly recurring reminder** to trigger an update check.
- The `customization/` folder is outside all upstream-managed paths and is safe across any update.
- Always run with a clean working tree; the skill creates a rollback backup branch + tag before touching anything.

---

## Additional Installation + Operations TODO (Requested)

This is the explicit implementation TODO list for this fork.

1. Signal CLI Integration
- Add a Signal channel skill (equivalent to /add-signal).
- Validate runtime dependencies for signal-cli.
- Add setup flow for linking a Signal number/device.
- Add health-check command for Signal connectivity.

2. GitHub Device Authentication Flow
- Add guided login flow for GitHub device auth in setup/customization.
- Persist auth status check (gh auth status or equivalent).
- Add remediation steps when token/session expires.

3. Backup to Repository in the same GitHub account
- Define backup target repo (this fork or dedicated backup repo).
- Add scheduled backup task (for example daily + weekly full snapshot).
- Backup scope should include:
  - shared PARA content
  - agent configuration
  - critical system logs/metadata
- Add restore playbook with validation checklist.

4. Repository Connection Workflow
- Add setup command to bind local workspace to target remote repository.
- Validate remote URL and write permissions.
- Create first baseline commit/tag for restore anchor.

5. Cross-agent Backup Governance
- Main Orchestrator owns backup policy and schedule.
- Support-ManagementConsultant audits backup success weekly.
- Alert path for backup failure to Executive-Suse and Executive-Andy.

6. Scientist Gap Closure
- Current local nanoclaw tree does not yet include `Agents/Support/Scientist/` under the family group.
- This fork must create and wire Support-Scientist explicitly for Suse workflows.
- Add delegation templates and output targets for talks, posters, manuscripts, and supervision tasks.

7. OneCLI Secret Assignment Validation
- During new agent-group setup, verify OneCLI secret mode and secret assignment before declaring setup complete.
- Add explicit setup check to avoid silent auth failures due to empty selective-mode assignments.

---

## Suggested Rollout Plan

Phase 1: Structure and contracts
1. Create all agent groups and assign scopes.
2. Create shared PARA directories and system files.
3. Install template files and handoff protocol.

Phase 2: Channel and auth
1. Implement Signal CLI installation flow.
2. Implement GitHub device auth helper flow.
3. Verify identity and permissions.

Phase 3: Backup automation
1. Connect target backup repository.
2. Add scheduled backup task.
3. Test backup restore on a clean path.

Phase 4: Operational hardening
1. Add weekly system report automation.
2. Add failure alerts and remediation runbooks.
3. Review role boundaries and write permissions.

---

## Definition of Done

This customization is complete when:
1. All Executive and Support subagents exist with clear scopes.
2. Shared PARA root exists with templates and governance docs.
3. Delegation protocol is used by all agents.
4. Signal integration setup is available.
5. GitHub auth workflow is documented and validated.
6. Automated backup to the chosen repository runs successfully.
7. Restore test succeeds and is documented.

---

## Notes for Future Extensions

Possible next additions:
- Per-agent quality scorecards.
- Automatic stale-project detection.
- Monthly archive automation.
- Policy checks before writing to protected files.
