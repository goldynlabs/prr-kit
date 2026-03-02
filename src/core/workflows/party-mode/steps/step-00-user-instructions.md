---
name: "step-00-user-instructions"
description: "Collect user instructions before the party review begins"
nextStepFile: "./step-01-load-reviewers.md"
---

# Step 0: User Instructions

## Goal
Always ask the user for scope, focus, requirements, or context before reviewers are loaded.
This step runs even if a `pr-context.yaml` already exists — fresh instructions override any prior `user_instructions` in the file.

## Sequence of Instructions

### 1. Show PR Summary

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎉 Party Mode — Review Setup

   Branch: {target_branch} → {base_branch}
   Files:  {file_count} changed
   PR:     {pr_number if set, else "local branch"}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 2. Prompt User

Display EXACTLY:

```
💬 Any instructions for this review?
   Press Enter to run a full standard review, or type your instructions below.

   You can specify:
     • Scope        "only security" / "security and architecture" / "skip performance"
     • Focus        "focus on SQL injection and rate limiting"
     • Requirements "all API endpoints must have auth middleware"
     • Context      "hotfix — ignore refactoring suggestions"
     • Mix freely   "security only, focus on JWT handling, context: auth rewrite in progress"
```

**HALT — wait for user response before continuing.**

### 3. Parse Response

**If user pressed Enter / left empty:**
- Set `user_instructions.provided` = `false`
- Set `user_instructions.review_scope` = `"all"`
- Set all other fields to `null`

**If user typed something**, parse the free-form text and extract:

**`review_scope`** — which reviews to run:
- Parse for scope signals: "only X", "just X", "X only", "skip X", "no X review", "X and Y"
- Map to codes: `GR` (general), `SR` (security), `PR` (performance), `AR` (architecture), `BR` (business)
- Examples:
  - "only security" → `[SR]`
  - "security and architecture" → `[SR, AR]`
  - "skip performance" → `[GR, SR, AR, BR]`
  - "focus on SQL injection" (no scope signal) → `"all"` (focus only, all reviewers still active)
- If no scope restriction found → `"all"`

**`focus_areas`** — specific things reviewers must prioritize (list of strings), or `null` if none.

**`custom_requirements`** — mandatory checks user specified (list of strings), or `null`.

**`context_notes`** — background info, trade-offs, constraints (list of strings), or `null`.

**`raw`** — full original text from user.

Set `user_instructions.provided` = `true`.

### 4. Acknowledge

**If user provided instructions:**

```
✅ Instructions captured.

   📋 Scope:        {scope_list joined with ", "  OR  "all reviewers"}
   🎯 Focus:        {focus_areas joined with ", "  OR  "standard coverage"}
   ✅ Requirements: {custom_requirements joined with ", "  OR  "none"}
   📝 Context:      {context_notes joined with "; "  OR  "none"}
```

**If user left empty:**

```
▶️  Full standard review — all reviewers, standard focus.
```

### 5. Write to pr-context.yaml

Write `user_instructions` to `{session_output}/pr-context.yaml`:

- **If `pr-context.yaml` exists**: update only the `user_instructions:` section, leave all other sections intact.
- **If `pr-context.yaml` does not exist** (Party Mode was run without prior DP/context collection):
  Create a minimal file with just `pr_metadata` and `user_instructions`:

```yaml
# PR-Specific Context (minimal — created by Party Mode)
# Generated: {timestamp}

pr_metadata:
  pr_number: {pr_number or null}
  branch: {target_branch}
  base_branch: {base_branch}
  collected_at: {ISO timestamp}

user_instructions:
  provided: {true|false}
  review_scope: {value}
  focus_areas: {value}
  custom_requirements: {value}
  context_notes: {value}
  raw: {value}
```

Store `pr_knowledge_base` = `{session_output}/pr-context.yaml` in working context.

### 6. Load Next Step

Add `step-00-user-instructions` to `stepsCompleted`. Load: `{nextStepFile}`
