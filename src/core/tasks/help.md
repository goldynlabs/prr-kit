# PR Review Help

## What to do next

Use `/prr-help` anytime for guidance on what to do.

### Typical Review Flow

**Per PR — Quick mode (1 command):**
- **[QR] Quick Review** — Full pipeline: select branch → describe → collect context → all 5 reviews → report → optional post

**Per PR — Manual mode (step by step):**
1. **[SP] Select PR** — Fetch latest, list branches/PRs, choose what to review
2. **[DP] Describe PR** — Auto-generate PR type, summary, file-by-file walkthrough
   ↳ *Context collected automatically — fresh, PR-specific, no setup needed*
3. **[GR/SR/PR/AR/BR] Review** — Run one or more specialized reviews
4. **[IC] Improve Code** — Get concrete code suggestions with inline fixes
5. **[AK] Ask** — Ask specific questions about the code changes
6. **[RR] Generate Report** — Compile all findings into a Markdown report
7. **[PC] Post Comments** — Post inline review comments to GitHub / GitLab / Azure DevOps / Bitbucket

### Available Reviews

- **[GR] General Reviewer** 👁️ — Logic, naming, readability, DRY, best practices, etc. — adapted to your stack
- **[SR] Security Reviewer** 🔒 — OWASP Top 10, injection, auth, secrets, etc. — adapted to your project
- **[PR] Performance Reviewer** ⚡ — N+1 queries, memory leaks, async patterns, etc. — adapted to your stack
- **[AR] Architecture Reviewer** 🏗️ — SOLID, layering, coupling, consistency, etc. — adapted to your architecture
- **[BR] Business Reviewer** 💼 — User impact, business risk, feature completeness, data safety, etc. — adapted to your project

### Finding Severity Levels

- 🔴 **[BLOCKER]** — Must fix before merge
- 🟡 **[WARNING]** — Should fix, with explanation
- 🟢 **[SUGGESTION]** — Nice-to-have improvement
- ❓ **[QUESTION]** — Cannot determine intent from diff — ask author before judging

### Utilities

- **[SS] Select Session** — List past review sessions and resume one. Use this at the start of a new conversation to pick up where you left off.
- **[CL] Clear** — Delete one or more past review session folders. Choose specific sessions or clear all.
