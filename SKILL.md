---
name: installer-runtime
description: Use this skill to run or resume a structured multi-phase installation guided by a pack spec and install-state.yaml. Triggers when the user wants to install, set up, or configure a system using an install pack — especially Google Workspace, GCP, or Apps Script. Also triggers for admin onboarding workflows, deployment contract setup, and any time a user mentions resuming, continuing, or checking where they left off in an installation. Recognizes intent even without technical vocabulary: "help me set up Workspace for my org", "where did I leave off", "continue my setup", "walk me through the install" are all strong triggers. Does NOT apply to one-off GCP console tasks, general scripting questions, or requests with no installation/onboarding context.
---

# Installer Runtime Skill

Orchestrates multi-phase installations using a declarative pack spec and persisted state.
The full execution loop is in `references/skill-body.md`. Load it immediately.
