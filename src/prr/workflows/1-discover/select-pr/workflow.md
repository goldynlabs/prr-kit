---
name: select-pr
description: "Fetch latest from remote, list branches/PRs, select target for review, load diff and confirm scope"
main_config: "{project-root}/_prr/prr/config.yaml"
nextStep: "./steps/step-01-fetch.md"
---

# Select PR Workflow

**Goal:** Clarify exactly which PR/branch to review by fetching latest from remote, listing available branches, letting the user select, loading the diff, and confirming scope before any review begins.

## WORKFLOW ARCHITECTURE

This uses **step-file architecture** with sequential orchestration:

- Step 01: `git fetch` to ensure latest state from remote
- Step 02: List branches sorted by recency, show commit summary
- Step 03: User selects branch, PR number, or enters commit range
- Step 04: Load full diff and commit log for selected PR
- Step 05: Show scope summary (files changed, lines +/-), confirm before proceeding

**Rules (NO EXCEPTIONS):**
- 🛑 NEVER start reviewing without completing this workflow first
- 📖 Read entire step file before executing
- ⏸️ ALWAYS halt at user selection points — never auto-select
- 💾 Create session folder `{session_output}` and store PR metadata in working context after confirmation (step 5)

## INITIALIZATION

Load config from `{main_config}` and resolve:
- `project_name`, `user_name`, `communication_language`
- `review_output`, `date` (system-generated)

Set `target_repo` = `.` (the repository containing this config file).

## EXECUTION

Read fully and follow: `{nextStep}`
