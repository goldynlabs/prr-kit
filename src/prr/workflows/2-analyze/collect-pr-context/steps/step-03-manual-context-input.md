---
name: "step-03-manual-context-input"
description: "Collect user instructions, requirements, and context for this review session"
nextStepFile: "./step-04-build-knowledge-base.md"
---

# Step 3: User Instructions & Context

## Goal
Let the user guide the review — they can restrict scope, set focus areas, add mandatory requirements, or provide background context. Their input directly controls which reviews run and what each reviewer prioritizes.

## Sequence of Instructions

### 1. Show Collection Summary

Print a brief summary of what was automatically collected so far:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Auto-collection complete.

📊 Collected:
   🗂️  Files changed:  {files_count}
   🎯 Domains:         {domains_list}
   🧩 Stacks:          {stacks_list or "none"}
   📘 Primary docs:    {primary_docs_found}
   ⚙️  Config files:   {config_files_found}
   💬 Annotations:     {annotations_count}
   🔌 External:        {mcp_and_rag_summary or "none"}
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

**IMPORTANT: Output this prompt as plain text only — do NOT use interactive question UI or structured choice tools. Wait for free-form text input.**

**HALT — wait for user response before continuing.**

### 3. Parse Response

**If user pressed Enter / left empty:**
- Set `user_instructions.provided` = `false`
- Set all fields to `null`

**If user typed something**, parse the free-form text and extract:

**`focus_areas`** — specific things reviewers must prioritize (list of strings), or `null` if none mentioned.

**`custom_requirements`** — mandatory checks user specified (list of strings), or `null`.

**`context_notes`** — background info, trade-offs, constraints (list of strings), or `null`.

**`raw`** — full original text from user.

Set `user_instructions.provided` = `true`.

### 4. Acknowledge

**If user provided instructions**, print a confirmation of what was parsed:

```
✅ Instructions captured — driving all downstream review steps.

   🎯 Focus:         {focus_areas joined with ", "  OR  "standard coverage"}
   ✅ Requirements:  {custom_requirements joined with ", "  OR  "none"}
   📝 Context:       {context_notes joined with "; "  OR  "none"}
```

**If user left empty:**

```
▶️  Full standard review — all reviewers, standard focus.
```

### 5. Load Next Step

Add `step-03-manual-context-input` to `stepsCompleted`. Load: `{nextStepFile}`
