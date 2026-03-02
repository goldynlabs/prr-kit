---
name: "step-04-output"
description: "Write PR description to file and display summary"
outputFile: "{session_output}/pr-description.md"
templateFile: "../templates/pr-description.template.md"
---

# Step 4: Output PR Description

## STEP GOAL

Compile all analysis into a structured PR description file and display summary to the reviewer.

## Sequence of Instructions

### 1. Generate PR Description

Using the template at `{templateFile}`, generate the PR description with:
- PR metadata (branch, base, PR number if applicable)
- PR type and risk level
- Executive summary (3-5 bullet points of key changes)
- File-by-file walkthrough
- Recommended reviews with rationale

### 2. Write to File

Write to `{outputFile}`.

### 3. Update Working Context

Store in working context:
- `pr_type` = `{classified_type}`
- `risk_level` = `{risk_level}`

### 4. Display Summary

Show the user:
```
✅ PR Description generated: {outputFile}

PR Type: {type} | Risk: {risk_level}
Summary:
{executive_summary_bullets}

Recommended next steps:
{recommended_reviews}

💡 Before reviewing: run [CC] Collect Context to build a fresh PR-specific knowledge base.
   Reviews use this context to apply your project's rules, stack-specific guidelines, and any
   external sources (Jira, Confluence, Figma) — skipping it means reviewers work with
   general rules only.
```

Workflow complete. Return to agent menu.
