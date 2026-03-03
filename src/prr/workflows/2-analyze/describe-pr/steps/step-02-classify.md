---
name: "step-02-classify"
description: "Classify PR type and determine review recommendations"
nextStepFile: "./step-03-walkthrough.md"
---

# Step 2: Classify PR

## STEP GOAL

Analyze the diff and commit messages to classify the PR type and generate review recommendations.

## Sequence of Instructions

### 1. Classify PR Type

Analyze the diff to determine the PRIMARY type (pick the best match):

| Type | Indicators |
|------|-----------|
| `bugfix` | Fix in logic, error handling, condition fix |
| `feature` | New files, new functions, new UI components |
| `refactor` | Same behavior, restructured code, renamed variables |
| `performance` | Caching, query optimization, async improvements |
| `security` | Auth changes, input validation, dependency security updates |
| `hotfix` | Urgent production fix, usually a targeted single change |
| `test` | Only test file changes |
| `docs` | Only documentation/comment changes |
| `config` | Config files, env, CI/CD changes |
| `chore` | Dependency updates, tooling, build changes |

### 2. Assess Risk Level

Based on PR type and what was changed:
- **🔴 High Risk**: Auth changes, payment logic, user data handling, security-critical paths
- **🟡 Medium Risk**: Core business logic, API changes, database schema
- **🟢 Low Risk**: UI tweaks, docs, test additions, minor refactors

### 3. Suggest Review Focus

Based on classification and risk level, suggest which reviews are most relevant — the orchestrator will use these as input alongside user instructions:
- bugfix → GR is core; SR if the fix touches security paths; BR if user-facing behavior changes
- feature → GR + AR to check design; PR if performance-sensitive changes; BR for user impact
- security → SR is highest priority; GR and BR for full picture
- performance → PR is core; GR for code quality; BR if user-facing impact
- refactor → AR to check structural consistency; GR for quality
- hotfix → fast GR + SR + BR to assess risk before shipping
- test/docs/config/chore → light GR or skip heavy reviews if changes are low-risk
- High-risk PRs → expand scope, don't skip SR or BR

These are starting suggestions — adapt based on what was actually changed and any user instructions.

### 4. Load Next Step

Add `step-02-classify` to `stepsCompleted`. Load: `{nextStepFile}`
