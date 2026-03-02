---
name: "step-05-confirm"
description: "Show PR scope summary and confirm before proceeding to review"
---

# Step 5: Confirm Scope

**Progress: Step 5 of 5** — Confirming review scope

## STEP GOAL

Show a clear summary of what will be reviewed, create the session output folder, store PR metadata in working context, and confirm the user wants to proceed.

## Sequence of Instructions

### 0. Compute Session Output Folder

Generate a unique, human-readable folder for this PR review session:

```
sanitized_branch = selected_branch
  .toLowerCase()
  .replace(/[^a-z0-9]+/g, '-')
  .replace(/^-+|-+$/g, '')
  .substring(0, 40)

if pr_number is set:
  session_slug = "pr{pr_number}-{sanitized_branch}"
else:
  session_slug = "{sanitized_branch}"

datetime_prefix = current datetime as "YYYY-MM-DD-HHmm"   (e.g. 2026-03-02-1430)

session_output = "{review_output}/{datetime_prefix}-{session_slug}"
```

Examples:
- `_prr-output/reviews/2026-03-02-1430-pr44-feature-auth-login`
- `_prr-output/reviews/2026-03-02-1430-feature-auth-login`

Create the folder:
```bash
mkdir -p "{session_output}"
```

**Store `session_output` in working context** — all subsequent workflows in this session will use it.

### 0b. Generate Diffs Folder

Parse the diff already loaded in memory and write per-file markdown files under `{session_output}/diffs/`, mirroring the repo folder tree.

**For each changed file in the diff:**

1. Determine file path, status, and line counts from the diff header.
2. Construct output path: `{session_output}/diffs/{file_path}.md`
3. Create parent directories:
   ```bash
   mkdir -p "{session_output}/diffs/{file_dir}"
   ```
4. Write the markdown file:

```markdown
# {file_path}
**Status:** {modified|added|deleted|renamed} | **+{lines_added} / -{lines_removed} lines**
---

**Line {start_line}** *(or **Lines {start}–{end}** for multi-line hunks)*
```diff
- old line content
+ new line content
```

*(repeat for each hunk in this file)*
```

**Rules:**
- **New file:** show all lines as `+` additions under a single hunk block.
- **Deleted file:** show all lines as `-` deletions under a single hunk block.
- **Renamed file with no content change:** write status line only, no diff blocks.
- **Renamed file with changes:** write status `renamed` and include change hunks as normal.
- One `.md` file per changed file. Folder structure mirrors the repo exactly.

**Example output for a modified file:**

```markdown
# src/auth/login.ts
**Status:** modified | **+45 / -12 lines**
---

**Line 23**
```diff
- const token = localStorage.getItem('token')
+ const token = cookieStorage.get('auth_token')
```

**Lines 67–70**
```diff
- async function login(user, pass) {
-   const result = await db.users.findOne(user)
+ async function login(user: string, pass: string): Promise<User> {
+   const result = await db.users.findOne({ email: user })
```
```

After writing all files, print:
```
✓ Diffs saved: {file_count} files → {session_output}/diffs/
```

### 1. Display Scope Summary

Present a clear summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PR Review Scope
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Target:  {selected_branch}
  Base:    {base_branch}
  PR #:    {pr_number}  (if applicable)
  Title:   {pr_title}  (if available)

📊 Diff Statistics:
  Commits: {commit_count}
  Files:   {files_changed_count}
{diff_stats}

⚠️  Review Strategy: {diff_strategy}
  (chunked = reviews will process file by file for accuracy)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Recommended reviews for this PR:
  [DP] Describe PR — understand changes before reviewing
  [GR] General Review — always recommended
  [SR] Security Review — if auth/API/user data touched
  [PR] Performance Review — if DB/async/frontend affected
  [AR] Architecture Review — if new patterns/structure added
  [BR] Business Review — for feature PRs, data migrations, or any high-risk change
```

### 2. Ask for Confirmation

```
✅ Ready to review. Which review would you like to run first?
   (Enter a command from the menu, or type 'DP' to start with Describe PR)
```

**HALT — wait for user response.** The workflow is complete. User returns to the agent menu.
