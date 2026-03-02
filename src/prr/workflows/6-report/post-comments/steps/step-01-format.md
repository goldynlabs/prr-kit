---
name: "step-01-format"
description: "Parse findings and build platform-specific comment payload"
nextStepFile: "./step-02-post.md"
---

# Step 1: Build Comment Payload

## Sequence of Instructions

### 1. Check Prerequisites

Verify working context contains:
- `pr_number` — if missing, show: `❌ No PR number found. Run [SS] Select Session or [SP] Select PR first.`
- `target_branch`, `base_branch`, `session_output`

Verify platform CLI is available:

| Platform | Check command | Error message |
|----------|--------------|---------------|
| github | `gh auth status` | `❌ Run: gh auth login` |
| gitlab | `glab auth status` | `❌ Run: glab auth login` |
| azure | `az account show` | `❌ Run: az login && az devops configure` |
| bitbucket | check env `BB_TOKEN` or `~/.config/bb` | `❌ Set Bitbucket credentials` |
| none | — | show: `ℹ️ No platform configured — findings saved to report only.` then stop |

**⚠️ GitHub CLI path detection** — `gh` may not be in `PATH` even if installed. If `gh auth status` fails, probe common locations before giving up:
```bash
# Windows
"/c/Program Files/GitHub CLI/gh" auth status
# macOS Homebrew
/opt/homebrew/bin/gh auth status
/usr/local/bin/gh auth status
# Linux
/usr/bin/gh auth status
~/.local/bin/gh auth status
```
Store the working path as `{gh_path}` (e.g. `"/c/Program Files/GitHub CLI/gh"`). Use `{gh_path}` for **all** subsequent `gh` calls in this workflow instead of bare `gh`.

### 2. Get Head Commit SHA

**GitHub / GitLab / Bitbucket:**
```bash
git -C {target_repo} rev-parse origin/{target_branch}
```
Or use `{pr_head_sha}` if already stored from select-pr step.

**Azure DevOps:**
```bash
az repos pr show --id {pr_number} --query lastMergeSourceCommit.commitId -o tsv
```

Store as `{commit_sha}`.

### 3. Load Review Report

Read the report at `{session_output}/final-review.md`.

### 4. Parse All Findings

Scan the report and extract every finding:
- `severity`: 🔴 Blocker / 🟡 Warning / 🟢 Suggestion / ❓ Question (also accept 📌 as alias for Question)
- `category`: security / performance / architecture / general / business
- `file_path`: relative path (e.g. `src/auth/login.js`) — null if not specified
- `line_number`: integer — null if not specified
- `description`: the issue text
- `suggested_fix`: fix/suggestion text

**Classify:**
- **Inline candidates**: `file_path` and `line_number` both present
- **Fallback findings**: no file or line reference

Limit inline comments to 30 max: include all 🔴 Blockers first, then 🟡 Warnings, then 🟢 Suggestions. Move overflow to fallback.

### 5. Format Comment Bodies

**🔴 Blocker:**
```
🔴 **[BLOCKER]** {emoji} {description}
**Risk:** {what could go wrong}
**Fix:** {suggested_fix}
```

**🟡 Warning:**
```
🟡 **[WARNING]** {emoji} {description}
**Why it matters:** {explanation}
**Suggestion:** {suggested_fix}
```

**🟢 Suggestion:**
```
🟢 **[SUGGESTION]** {description}
{suggested_fix}
```

Category emojis: 🔒 security · ⚡ performance · 🏗️ architecture · 👁️ general · 💼 business

### 6. Build Summary Body

```markdown
## 🔍 AI Code Review

**PR:** `{target_branch}` → `{base_branch}` | **Date:** {date} | **Reviewer:** {user_name}

**Verdict:** {verdict}

| Category | 🔴 | 🟡 | 🟢 |
|----------|----|----|----|
| 🔒 Security | {n} | {n} | {n} |
| ⚡ Performance | {n} | {n} | {n} |
| 🏗️ Architecture | {n} | {n} | {n} |
| 👁️ General | {n} | {n} | {n} |
| 💼 Business | {n} | {n} | {n} |

### Executive Summary
{bullets}

{fallback_findings_section}

❓ {question_count} questions for author

---
*{inline_count} inline comments on code lines*
```

Verdict logic: any 🔴 → `REQUEST_CHANGES` / only 🟡 → `COMMENT` / none → `APPROVE`

### 7. Build Platform Payload

**⚠️ CRITICAL — Use Write tool + runtime script. NEVER use bash heredoc.**

Bash heredoc breaks with Unicode characters, emojis, backticks, and nested quotes — all of which appear in review comment bodies. The only reliable approach is:

1. **Write** the build script to `{temp_dir}/build-payload.mjs` using the Write tool
2. **Execute** it with `node`
3. Script writes the final JSON to `{temp_dir}/prr-payload.json`

**Step A — Detect available runtime:**
```bash
node --version 2>/dev/null && echo "use-node" || (python3 --version 2>/dev/null && echo "use-python3") || echo "no-runtime"
```
Prefer `node`. Fallback: `python3`, then `python`.

**Step B — Write the build script using the Write tool** (NOT echo/heredoc/Bash):

Use the Write tool to create `{temp_dir}/build-payload.mjs` (Node.js) or `{temp_dir}/build-payload.py` (Python).

The script must:
- Define all string values (comment bodies, summary) as native string variables — no manual JSON escaping needed, the runtime handles it
- Build the payload object in memory
- Write to `{temp_dir}/prr-payload.json` using native JSON serialization:
  - Node.js: `JSON.stringify(payload, null, 2)`
  - Python: `json.dumps(payload, ensure_ascii=False, indent=2)`

**Node.js template** (`build-payload.mjs`):
```js
import { writeFileSync } from 'fs'

const payload = {
  commit_id: "COMMIT_SHA",
  body: "SUMMARY BODY — full markdown here, no escaping needed",
  event: "REQUEST_CHANGES", // or "COMMENT" or "APPROVE"
  comments: [
    { path: "src/file.js", line: 42, side: "RIGHT", body: "🔴 **[BLOCKER]** ..." },
    // ... more comments
  ]
}

writeFileSync("{temp_dir}/prr-payload.json", JSON.stringify(payload, null, 2), "utf-8")
console.log(`OK: ${payload.comments.length} comments`)
```

**Python template** (`build-payload.py`):
```python
import json

payload = {
    "commit_id": "COMMIT_SHA",
    "body": "SUMMARY BODY",
    "event": "REQUEST_CHANGES",
    "comments": [
        {"path": "src/file.js", "line": 42, "side": "RIGHT", "body": "🔴 **[BLOCKER]** ..."},
    ]
}

with open("{temp_dir}/prr-payload.json", "w", encoding="utf-8") as f:
    json.dump(payload, f, ensure_ascii=False, indent=2)
print(f"OK: {len(payload['comments'])} comments")
```

**Step C — Execute:**
```bash
node "{temp_dir}/build-payload.mjs"
# or: python3 "{temp_dir}/build-payload.py"
```

**Step D — Verify output (optional sanity check):**
```bash
# Node.js
node -e "const p=require('{temp_dir}/prr-payload.json'); console.log('OK:', p.comments.length, 'comments')"
# Python
python3 -c "import json; p=json.load(open('{temp_dir}/prr-payload.json')); print('OK:', len(p['comments']), 'comments')"
```

---

**Payload schema reference:**

**GitHub** — `{temp_dir}/prr-payload.json`:
```json
{
  "commit_id": "{commit_sha}",
  "body": "{summary_body}",
  "event": "REQUEST_CHANGES|COMMENT|APPROVE",
  "comments": [
    { "path": "{file_path}", "line": {line_number}, "side": "RIGHT", "body": "{body}" }
  ]
}
```

**GitLab** — one file per comment `{temp_dir}/prr-payload-{n}.json` + `{temp_dir}/prr-summary.md`:
```json
{ "body": "{body}", "position": { "base_sha": "{base_sha}", "head_sha": "{commit_sha}",
  "start_sha": "{base_sha}", "position_type": "text",
  "new_path": "{file_path}", "new_line": {line_number} } }
```

**Azure DevOps** — `{temp_dir}/prr-thread-{n}.json` per comment:
```json
{ "comments": [{ "parentCommentId": 0, "content": "{body}", "commentType": 1 }],
  "threadContext": { "filePath": "{file_path}",
    "rightFileStart": { "line": {line_number}, "offset": 1 },
    "rightFileEnd": { "line": {line_number}, "offset": 1 } },
  "status": "active" }
```

**Bitbucket** — `{temp_dir}/prr-bb-{n}.json` per comment:
```json
{ "content": { "raw": "{body}" },
  "inline": { "to": {line_number}, "path": "{file_path}" } }
```

### 8. Display Preview

```
📋 Payload ready:
  Platform:          {active_platform}
  Inline comments:   {inline_count}
  Fallback comments: {fallback_count}
  Verdict:           {event}
```

### 9. Load Next Step

Add `step-01-format` to `stepsCompleted`. Load: `{nextStepFile}`
