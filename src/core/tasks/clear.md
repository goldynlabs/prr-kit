# Clear PRR Output

**Goal:** Remove prr-kit context files and/or review reports from the output folder. Useful when starting fresh, switching projects, or cleaning up after a review session.

## EXECUTION

### 1. Load Config

Read `{project-root}/_prr/prr/config.yaml` to get `review_output` path.

### 2. Scan Session Folders

List all session folders in `{review_output}` (datetime-slug subdirectories):

```bash
ls -d "{review_output}"/*/
```

Each session folder is named `{YYYY-MM-DD-HHmm}-{slug}` (e.g. `2026-03-02-1430-pr44-feature-auth`).
Inside each folder are: `pr-context.yaml` (knowledge base) and review `.md` files.

Also detect any legacy flat files still directly in `{review_output}/` (pre-session-folder format).

### 3. Display What Exists

Show the user a summary of each session:

```
🗂️ PRR Output — {review_output}

Session folders ({n}):

  📁 2026-03-02-1430-pr44-feature-auth/
     Branch: feature/auth  |  PR #44  |  2026-03-02
     Files: pr-context.yaml, pr-description.md,
            general-review.md, security-review.md, final-review.md

  📁 2026-03-01-0915-fix-null-pointer/
     Branch: fix/null-pointer  |  no PR  |  2026-03-01
     Files: general-review.md

  ... (list all session folders)
```

If no session folders exist:
```
✅ Nothing to clear — output folder is already empty.
```
Then stop.

### 4. Ask Scope

Ask the user what to clear:

```
What would you like to clear?

  [1] All sessions      — delete all session folders
  [2] Select sessions   — choose specific session(s) to delete
  [3] Cancel            — do nothing
```

Wait for user input.

**If [2] Select sessions:** List sessions numbered and ask:
```
Which session(s) to delete? (e.g. 1, 3 or 1-3)
```
Wait for response.

### 5. Execute Deletion

**If [1] All sessions:**
```bash
rm -rf "{review_output}"/*/
```

**If [2] Selected sessions:** For each chosen session folder:
```bash
rm -rf "{review_output}/{selected_session_folder}/"
```

**If [3] Cancel:**
```
❌ Cancelled — nothing was deleted.
```
Stop.

### 6. Confirm Deletion

Verify folders are gone and report:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Clear complete

Deleted:
  🗑️  Sessions removed: {n} folder(s)

Output folder: {review_output}
Status: clean

Ready for a fresh review session.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
