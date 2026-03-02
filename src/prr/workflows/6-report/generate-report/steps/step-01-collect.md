---
name: "step-01-collect"
description: "Collect findings from all completed review files"
nextStepFile: "./step-02-organize.md"
---

# Step 1: Collect All Findings

## Sequence of Instructions

### 1. Detect Completed Reviews

Check which review files exist in `{session_output}/`:

```bash
ls "{session_output}/"
```

### 2. Collect Review Output Files

For each review file that exists, read it:
- `general-review` → `{session_output}/general-review.md`
- `security-review` → `{session_output}/security-review.md`
- `performance-review` → `{session_output}/performance-review.md`
- `architecture-review` → `{session_output}/architecture-review.md`
- `business-review` → `{session_output}/business-review.md`
- `improve-code` → `{session_output}/improve-code.md`

### 3. Parse All Findings

From each file, extract all findings with their:
- Severity (🔴 Blocker / 🟡 Warning / 🟢 Suggestion / ❓ Question / Critical/High/Medium/Low)
- Category (general/security/performance/architecture/business/improvement)
- File + line reference
- Description
- Suggested fix

**Severity mapping:**
- `🔴` or `Critical`/`High` → Blocker
- `🟡` or `Medium` → Warning
- `🟢` or `Low` → Suggestion
- `❓` → Question (collect separately — do not count as blocker/warning/suggestion)

### 4. Count Statistics

Count:
- Total findings by severity
- Findings by category
- Files with issues

### 5. Load Next Step

Add `step-01-collect` to `stepsCompleted`. Load: `{nextStepFile}`
