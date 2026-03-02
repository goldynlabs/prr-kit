# Select Session

**Goal:** List all past PR review sessions and let the user resume one by restoring `session_output` into working context.

## EXECUTION

### 1. Load Config

Read `{project-root}/_prr/prr/config.yaml` to get `review_output` path.

### 2. Scan Session Folders

List all session folders in `{review_output}`:

```bash
ls -d "{review_output}"/*/
```

For each session folder, read its contents to build a summary:

```bash
ls "{review_output}/{session_folder}/"
```

From the files present, determine:
- **Branch / PR** — from folder name (e.g. `2026-03-02-1430-pr44-feature-auth` → PR #44, branch `feature/auth`)
- **Date / Time** — from folder name prefix (`2026-03-02-1430`)
- **Reviews done** — which `*-review.md` files exist
- **Has final report** — `final-review.md` exists
- **Has description** — `pr-description.md` exists

### 3. Display Sessions

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📂 Review Sessions — {review_output}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  [1] 2026-03-02 14:30 — PR #44  feature/auth-login
      Reviews: DP ✓  GR ✓  SR ✓  RR ✓
      Report:  final-review.md ✓

  [2] 2026-03-01 09:15 — fix/null-pointer-checkout
      Reviews: DP ✓  GR ✓
      Report:  —

  [3] 2026-02-28 17:00 — PR #41  chore/upgrade-deps
      Reviews: DP ✓  GR ✓  AR ✓  RR ✓
      Report:  final-review.md ✓

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If no sessions exist:
```
✅ No sessions found in {review_output}.
   Run [SP] Select PR or [QR] Quick Review to start a new session.
```
Then stop.

### 4. Ask User to Select

```
Select a session to resume (enter number):
```

**HALT — wait for user response.**

### 5. Restore Session

Set `session_output` = full path of the selected session folder.

Load PR metadata from working context sources available in the session folder:
- Parse `session_output` folder name to extract: `datetime_prefix`, `pr_number` (if `prN-` prefix), `branch_slug`
- Read `pr-context.yaml` if it exists — extract `pr_metadata.pr_number`, `pr_metadata.branch`, `pr_metadata.base_branch`
- Derive `target_branch` and `base_branch` from knowledge base or folder name

**Store in working context:**
- `session_output` = selected folder path
- `target_branch`, `base_branch`, `pr_number` = from above

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Session restored

📁 {session_output}
   Branch: {target_branch} → {base_branch}
   PR #:   {pr_number}  (if applicable)

Reviews completed: {list or "none"}
Ready to continue — run any command from the menu.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
