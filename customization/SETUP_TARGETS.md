# NanoClaw Setup Targets

## Purpose
This file captures the concrete setup work requested for this fork.

## Local Fork Prerequisites (must be true first)
- Runtime baseline: Docker available, Node.js 20+, pnpm 10+, and Windows usage via WSL2 path for NanoClaw runtime.
- OneCLI baseline: credentials are managed through OneCLI; agent sessions need valid secret assignment.
- Family group container baseline in local fork includes `python3-pip` in `groups/family/container.json`.
- Keep `groups/family/CLAUDE.local.md` as the authoritative family policy overlay.

## Critical OneCLI Gotcha
- Newly auto-created agents can start in `selective` secret mode with no assigned secrets.
- Setup must include explicit verification and remediation:
  - `onecli agents list`
  - `onecli agents secrets --id <agent-id>`
  - either `onecli agents set-secret-mode --id <agent-id> --mode all`
  - or explicit secret assignment using `onecli agents set-secrets --id <agent-id> --secret-ids ...`

## 1) Signal CLI Integration

### Goal
Enable Signal as an operational channel in NanoClaw via signal-cli.

### Requirements
- Install and validate signal-cli runtime dependencies.
- Add channel integration flow for Signal in setup/customization.
- Register and link a Signal account/device.
- Add a health-check command for Signal connectivity and send/receive test.
- Add channel-level diagnostics command that reports:
  - signal-cli availability/version
  - account link status
  - last inbound/outbound delivery status
  - remediation hints for common failure states

### Definition of Done
- Signal channel can be installed from this fork.
- A test message can be received and replied to by the configured agent.
- Failure states (not linked, expired session, missing dependency) return actionable remediation.

---

## 2) Voice Message Processing

### Goal
Allow agents to process incoming voice messages and respond with usable output.

### Requirements
- Detect voice/audio messages from supported channels (starting with Signal path).
- Transcribe voice to text.
- Pass transcript into normal agent workflow.
- Preserve transcript and message metadata in session history.
- Define fallback behavior when transcription fails.
- Add language handling (at least de/en auto-detection or configurable default).
- Define storage policy for audio artifacts (store metadata by default; retain raw audio only when explicitly configured).

### Optional Extension
- Add text-to-speech response mode for selected agents/channels.

### Definition of Done
- Voice message input results in a transcript-backed agent response.
- Transcription errors are logged and user-facing failure message is clear.
- Behavior is documented for both happy path and error path.

---

## 3) Calendar Integration

### Goal
Keep calendar workflows fully operational via the Kalender agent and CalDAV.

### Requirements
- Keep Kalender as the sole calendar integration interface for other agents.
- Ensure CalDAV sync scripts are usable in this fork.
- Ensure credentials are read from environment and never committed.
- Ensure sync output populates family calendar files used by Executive agents.
- Ensure conflict detection and escalation behavior to Executive-Suse and Executive-Andy.
- Enforce required env vars:
  - `POSTEO_USERNAME`
  - `POSTEO_PASSWORD`
  - optional `DAYS_AHEAD` (default 30)
- Preserve current script capabilities:
  - `sync_calendar.py` writes `Areas/Family/kalender.md` plus per-person files (`kalender-andi.md`, `kalender-suse.md`, `kalender-familie.md`, `kalender-felix.md`, `kalender-marie.md`, `kalender-rosa.md`)
  - `import_ics.py` supports `--overview`, `--import`, and `--reimport`

### Existing Local Script Baseline (reference)
- sync_calendar.py (CalDAV -> markdown sync)
- import_ics.py (ICS -> Posteo family calendar import)

### Operations Notes
- Manual sync command baseline: `python Agents/Support/Kalender/sync_calendar.py`
- ICS import command baseline: `python Agents/Support/Kalender/import_ics.py --import`
- Dependencies baseline: `caldav`, `icalendar`, `python-dotenv`

### Definition of Done
- Manual sync run succeeds with valid credentials.
- Calendar files are updated and consumable by agent workflows.
- Conflict/escalation logic is documented and testable.
- Kalender workflow can run end-to-end without other agents touching CalDAV directly.

---

## 4) Repository Backup

### Goal
Back up agents, PARA structure, and all customizations to a Git repository.

### Backup Scope
- groups/family/Agents/ (Executive + Support instructions and local agent assets)
- groups/family/Projects/
- groups/family/Areas/
- groups/family/Resources/
- groups/family/Archives/
- groups/family/container.json
- groups/family/CLAUDE.local.md
- customization/
- Relevant setup/customization scripts and policy docs

### Requirements
- Use a dedicated backup repository (or explicitly approved target repository).
- Add automated backup schedule (recommended: daily incremental, weekly full checkpoint tag).
- Add integrity checks after each backup run.
- Add restore workflow with verification checklist.
- Include backup status artifact with:
  - source revision
  - backup revision
  - included paths summary
  - exclusion summary

### Safety Rules
- Never commit secrets (.env, tokens, credentials, private keys).
- Maintain .gitignore coverage for sensitive files.
- Log backup result (success/failure, timestamp, commit hash/tag).
- Explicitly exclude:
  - `.env`
  - `repo-tokens/`
  - runtime data/log directories
  - any file containing raw auth credentials

### Definition of Done
- Scheduled backups run without manual intervention.
- Restore test to a clean path succeeds.
- Recovery steps are documented and reproducible.

---

## 5) Upstream NanoClaw Updates

### Goal
Keep this fork current with upstream security fixes and improvements without losing customizations.

### How it works
NanoClaw ships a built-in `/update-nanoclaw` skill that handles this on demand:
- Creates a timestamped backup branch/tag before any change
- Shows a preview of upstream changes grouped by category (skills, src, config)
- Offers merge, cherry-pick, or rebase
- Runs `pnpm build` + `pnpm test` after merge
- Checks CHANGELOG for `[BREAKING]` entries and triggers migration skills if needed

The `customization/` folder is outside all upstream-managed paths, so it is safe across any update.

### Requirements
- After deployment, schedule a recurring reminder for the Orchestrator to run `/update-nanoclaw` (recommended: weekly).
- Verify `upstream` remote points to `https://github.com/qwibitai/nanoclaw.git` on first run.
- Always ensure working tree is clean before running (commit or stash first).
- Review CHANGELOG for `[BREAKING]` entries after each update.

### Rollback
```
git reset --hard pre-update-<hash>-<timestamp>
```
Backup branch also available under `backup/pre-update-<hash>-<timestamp>`.

### Definition of Done
- `upstream` remote is configured and reachable.
- First `/update-nanoclaw` run completes without conflict.
- Weekly update reminder is active in Orchestrator recurrence schedule.

---

## Execution Priority
1. Signal CLI Integration
2. Voice Message Processing
3. Calendar Integration hardening
4. Repository Backup automation
5. Upstream update schedule

## Owner Model
- Main Orchestrator: owns delivery tracking, status reporting, and weekly update reminders.
- Support-Kalender: owns calendar integration correctness.
- Support-ManagementConsultant: audits backup reliability and operational quality.
- Executive-Suse and Executive-Andy: final approval for production rollout.
