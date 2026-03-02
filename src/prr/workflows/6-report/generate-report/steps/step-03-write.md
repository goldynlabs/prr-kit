---
name: "step-03-write"
description: "Write the final review report to file"
outputFile: "{session_output}/final-review.md"
templateFile: "../templates/review-report.template.md"
---

# Step 3: Write Report

## Sequence of Instructions

### 1. Generate Report

Using `{templateFile}`, write the complete review report to `{outputFile}`.

The report includes:
- PR metadata and review summary
- Executive summary with technical verdict + business risk level
- Business Impact section (if business-review was completed)
- All findings organized by severity and category
- Files with multiple issues highlighted
- Recommended actions (must-fix vs nice-to-have)
- Post-ship monitoring checklist (if business-review completed)

### 2. Display Completion

```
✅ Review Report Generated!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📄 File: {session_output}/final-review.md

Summary:
  🔴 Blockers: {blocker_count}
  🟡 Warnings: {warning_count}
  🟢 Suggestions: {suggestion_count}
  📌 Questions: {question_count}

Verdict: {verdict}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Next steps:
  [PC] Post Comments — post these findings to GitHub / GitLab / Azure / Bitbucket PR/MR
  or share the report file directly with the PR author
```

**Workflow complete.** Return to agent menu.
