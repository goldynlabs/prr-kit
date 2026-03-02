---
name: party-mode
description: "Multi-reviewer discussion: all specialized agents review and debate the PR together"
main_config: "{project-root}/_prr/prr/config.yaml"
nextStep: "./steps/step-00-user-instructions.md"
---

# Party Mode Workflow 🎉

**Goal:** Simulate a multi-reviewer code review session where all specialized agents (General, Security, Performance, Architecture, Business) each contribute their perspective on the PR, then debate any conflicting findings.

## WORKFLOW ARCHITECTURE

3-step process:
1. Collect user instructions (scope, focus, requirements, context) — always runs, user must respond
2. Load reviewer personas filtered by scope
3. Run structured discussion with each active reviewer contributing findings

## WHEN TO USE

Use Party Mode when you want:
- A comprehensive review from all angles in one session
- Reviewers to challenge each other's findings
- A realistic team code review feel
- Faster than running all 5 reviews separately

## INITIALIZATION

Load config from `{main_config}`.
Use `session_output`, `target_branch`, `base_branch`, `pr_number` from working context.
If not set: prompt user to run [SS] Select Session or [SP] Select PR first.

## EXECUTION

Read fully and follow: `{nextStep}`
