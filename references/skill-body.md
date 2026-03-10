# Installer Runtime Skill

This file is the execution skill for Claude. When a user asks you to run an installation
using a pack (e.g. "help me install Google Workspace and Apps Script"), load this file
and follow the instructions below.

---

## Session Start

1. Locate the pack spec: `packs/<pack-id>/install-spec.yaml`, or as directed by the user.
2. Load `install-state.yaml` from the working directory. If absent, create it:

   ```yaml
   current_phase: <first phase id from pack spec>
   status: running
   facts: {}
   evidence: []
   phase_confirmations: {}
   ```

3. Determine the resume point (see Resume Logic below) and announce it.
4. If the user said "walkthrough mode", use walkthrough mode for this session (no Bash,
   synthetic probe results). Otherwise use live mode (real Bash commands).

---

## Resume Logic

Find the **earliest phase** in `pack.phases` (in order) where either condition holds:

- Not all `phase.required_facts` are present as keys in `state.facts`, OR
- No evidence record exists with `item_id == "transition:<phase_id>"`,
  `item_type == "manual"`, `result == "pass"`

Begin there. Do not re-run probes for phases that already have complete transition evidence.

---

## Phase Loop

Repeat for the current phase until `state.status == "done"`:

### 1. Run probes

For each probe in `phase.probe_ids`:

- **`kind: cli` or `kind: api` (live mode)** — run the Bash/API command, parse output,
  set facts from `probe.updates_facts`.
- **`kind: manual` (any mode)** — ask the operator to confirm or provide the value.
  Do not advance until they respond.
- **Walkthrough mode (any kind)** — describe what the probe would do, write a synthetic
  `pass` result, set facts as if it passed. Still ask the operator for `kind: manual`.

After each probe, append an evidence record and write `install-state.yaml`.

For `failure_class: transient` failures, retry with backoff: 10s, 20s, 40s, 80s, 120s max.
Record each attempt in evidence. For `failure_class: hard` failures, stop and report.

### 2. Run eligible actions

For each action in `phase.action_ids`:

- Evaluate `action.preconditions` against current facts. Skip if not satisfied.
- Apply the confirmation policy:

  | Type | Confirmation required |
  |---|---|
  | `suggest` | None |
  | `mutate_safe` | Once per phase (batch all `mutate_safe` in the phase together) |
  | `mutate_sensitive` | Explicit per action |
  | `manual` | Explicit; operator must acknowledge before evidence is written |

- Live mode: execute. Walkthrough mode: describe, ask for confirmation as normal,
  write synthetic pass evidence.

After each action, update facts from `action.effects`, append evidence, write
`install-state.yaml`.

### 3. Evaluate transition

- Check all `phase.required_facts` are present.
- Evaluate `phase.transition_when` against current facts.
- If satisfied, write a transition evidence record and advance to the next phase:

  ```yaml
  ts: <ISO 8601 timestamp>
  phase: <phase_id>
  item_id: transition:<phase_id>
  item_type: manual
  result: pass
  summary: "Transition condition satisfied"
  ```

- If not satisfied, report which facts or conditions are missing. Ask the operator:
  retry probes, skip with justification, or abort.
- If this was the last phase, set `state.status: done` and report completion.

---

## Evidence Format

Every probe result, action result, and transition produces one record. All six fields
are required (validated by `schemas.js:validateEvidenceRecord`):

```yaml
ts: <ISO 8601 timestamp>
phase: <phase_id>
item_id: <probe_id | action_id | "transition:<phase_id>">
item_type: <probe | action | manual>
result: <pass | fail | unknown>
summary: <one-line description>
```

Write `install-state.yaml` after every record. Never skip a write — this is what makes
resume safe across sessions.

---

## State File Format

Only two `status` values are valid: `running` (in progress) and `done` (all phases complete). Never write `complete`, `finished`, or any other value.

```yaml
current_phase: <phase_id>
status: running | done   # ONLY these two values — never "complete" or anything else
facts:
  <fact_id>: <value>
evidence:
  - ts: ...
    phase: ...
    item_id: ...
    item_type: ...
    result: ...
    summary: ...
phase_confirmations:
  <phase_id>:
    mutate_safe: true
```

---

## Repair Paths

If a phase cannot progress, consult `pack.repair_paths.<phase_id>` for the documented
recovery steps. Apply them in order. Record each attempt in evidence. If all repair
paths are exhausted, record `unknown` and ask the operator for direction.

---

## Example Invocation

> "Using the install-pack-google-workspace-apps-script pack, help me install
> Google Workspace and Apps Script."

Load `packs/google-workspace-gcp-apps-script/install-spec.yaml`, load or create
`install-state.yaml`, announce the resume point, and begin the phase loop.
