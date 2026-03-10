---
name: installer-runtime
description: Orchestrates multi-phase installations using a pack spec (install-spec.yaml) and persisted state (install-state.yaml). Use this skill whenever a user wants to install, set up, configure, or resume a structured installation — especially for Google Workspace, GCP, or Apps Script setups. Also use when the user mentions resuming an installation, running phases, or any workflow involving probes, checkpoints, and state persistence. Use even if the user doesn't say "skill" explicitly — phrases like "help me install", "set up my GCP project", "continue my installation", or "where did I leave off" are all strong triggers.
---

# Installer Runtime Skill

Orchestrates multi-phase installations using a declarative pack spec and persisted state.
The full execution loop is in `references/skill-body.md`. Load it immediately.
