---
name: "step-04-build-knowledge-base"
description: "Build structured PR-specific knowledge base for reviewers"
---

# Step 4: Build PR-Specific Knowledge Base

## Goal
Transform collected data into structured knowledge base optimized for reviewers.

## Sequence of Instructions

### 1. Announce Knowledge Base Building

```
🧠 Building PR-specific knowledge base...
```

### 2. Map `manual_context` → `user_instructions`

Using the `manual_context` variable set in step-03:

- **If `manual_context` is null** (user skipped): set `user_instructions.provided = false`, all other fields `null`.
- **If `manual_context` is not null** (user provided text): set `user_instructions.provided = true`, `raw = manual_context`, then parse the free-form text:
  - `review_scope` — look for scope signals ("only SR", "skip performance", "security and architecture"). Default `"all"` if none found.
  - `focus_areas` — extract specific things to prioritize (e.g. "focus on JWT handling" → `["JWT handling"]`). `null` if none.
  - `custom_requirements` — extract mandatory checks the user stated (e.g. "all endpoints must have auth middleware"). `null` if none.
  - `context_notes` — extract background info, trade-offs, constraints. `null` if none.

### 3. Create Knowledge Base Structure

Build YAML structure with all collected context:

```yaml
# PR-Specific Context
# Generated: {timestamp}
# For PR: {pr_number} / Branch: {branch_name}

pr_metadata:
  pr_number: {pr_number}
  branch: {branch_name}
  base_branch: {base_branch}
  files_changed: {n}
  collected_at: {ISO timestamp}

# ⚠️  IMPORTANT — Human-provided instructions from the user.
# All reviewers MUST read this section before starting any review.
# review_scope controls which reviews run. focus_areas, custom_requirements, and context_notes
# are highest-priority guidance — align all findings against them.
user_instructions:
  # Populated only when the user provided input in step-03-manual-context-input.
  # If provided: true — treat this content as the highest-priority context in this file.
  provided: {true|false}
  review_scope: {"all" | [SR] | [SR, AR] | ...}   # "all" = run all reviews; list = only those codes
  focus_areas: {list of strings, or null}           # specific things every reviewer must prioritize
  custom_requirements: {list of strings, or null}   # mandatory checks the user specified
  context_notes: {list of strings, or null}         # background info, trade-offs, constraints
  raw: {original text from user, or null}           # verbatim input

files_analysis:
  changed_files:
    - path: src/stores/todoStore.js
      extension: .js
      category: pinia-store
      domains: [state-management]

    - path: src/views/TodoListView.vue
      extension: .vue
      category: vue-view
      domains: [ui-components, state-management]

  domains_involved:
    - state-management
    - ui-components

  file_types:
    - .js (1 file)
    - .vue (1 file)

relevant_rules:
  # ESLint rules that apply to this PR
  eslint:
    vue/multi-word-component-names:
      severity: error
      applies_to: [TodoListView.vue]
      description: Component names must be multi-word

    vue/require-prop-types:
      severity: error
      applies_to: [*.vue]
      description: Props must have type definitions

    prefer-const:
      severity: error
      applies_to: [*.js, *.vue]
      description: Use const instead of let when variable is not reassigned

    no-var:
      severity: error
      applies_to: [*.js, *.vue]
      description: No var keyword allowed

  prettier:
    semi: false
    singleQuote: true
    tabWidth: 2
    printWidth: 100
    applies_to: all_files

relevant_guidelines:
  # From CLAUDE.md
  state_management:
    source: CLAUDE.md
    section: State Management
    content: |
      Use Pinia stores with setup function style.
      - State: use ref() and reactive()
      - Getters: use computed()
      - Actions: regular functions
      - No Options API allowed

    applies_to: [src/stores/todoStore.js]

  security:
    source: CLAUDE.md
    section: Security
    content: |
      Never use v-html with user input.
      All inputs must be validated and sanitized.

    applies_to: [src/views/TodoListView.vue]

  # From CONTRIBUTING.md
  component_standards:
    source: CONTRIBUTING.md
    section: Component Standards
    content: |
      - Use PascalCase for component names
      - Multi-word names required (enforced by ESLint)
      - Props must have types and defaults
      - Use Composition API with <script setup>

    applies_to: [src/views/TodoListView.vue]

  # From ARCHITECTURE.md
  container_presentational_pattern:
    source: ARCHITECTURE.md
    section: Component Architecture
    content: |
      Follow Container/Presentational pattern:
      - Container (views/): Connect to stores, handle logic
      - Presentational (components/): Receive props, emit events

    applies_to: [src/views/TodoListView.vue]

  pinia_patterns:
    source: ARCHITECTURE.md
    section: State Management
    adr: ADR-002
    content: |
      Use Pinia with setup function style.
      Rationale: Simpler API, better TypeScript support.
      Actions modify state directly (no mutations).

    applies_to: [src/stores/todoStore.js]

inline_context:
  annotations:
    - file: src/stores/todoStore.js
      line: 10
      type: "@pattern"
      content: "Use composition API only"
      priority: high

    - file: src/stores/todoStore.js
      line: 15
      type: "@security"
      content: "Validate all inputs before storage"
      priority: critical

stack_context:
  # Populated from data/stacks/{stack}.md files for each detected stack.
  # All reviewers (GR, SR, PR, AR, BR) MUST read and apply these rules
  # in addition to project-specific guidelines.
  detected_stacks: [vue3, typescript]

  rules:
    vue3:
      security:
        - severity: critical
          rule: "v-html with user-controlled data → XSS. Use {{ }} or DOMPurify."
        - severity: high
          rule: "Dynamic :is binding with user string → arbitrary component injection."
      performance:
        - severity: high
          rule: "watchEffect/watch with async ops without onCleanup → memory leak on unmount."
        - severity: medium
          rule: "v-for without :key or using index key → broken reconciliation on reorder."
      architecture:
        - severity: high
          rule: "Direct store state mutation outside action → bypasses Pinia devtools."
        - severity: high
          rule: "Props mutation instead of emit → violates one-way data flow."
      code_quality:
        - severity: high
          rule: "defineProps without TypeScript types → silent prop misuse."
        - severity: medium
          rule: "Options API mixed with Composition API in same component."
      common_bugs:
        - severity: high
          rule: "reactive() destructuring loses reactivity — use toRefs() to preserve it."
        - severity: medium
          rule: "watch missing immediate:true when logic should run on initial value."
    typescript:
      architecture:
        - severity: high
          rule: "'any' type defeats TypeScript purpose — use unknown + type narrowing."
        - severity: high
          rule: "strict:false or missing strict in tsconfig → implicit any, loose null checks."
      code_quality:
        - severity: high
          rule: "@ts-ignore without explanation hides real bugs."
        - severity: medium
          rule: "Missing return type annotation on exported functions."
    # ... additional detected stacks follow same pattern

external_context:
  # Populated only when external_sources.enabled: true and tools were available
  mcp_tools_used: []              # e.g. ["confluence", "jira"]

  knowledge_base:                 # From Confluence / Notion / etc.
    - source: Confluence
      page: "Engineering Standards"
      content: |
        All state mutations must be logged for audit purposes.
        JWT tokens must be validated server-side.

  issue_context:                  # From Jira / Linear / GitHub Issues
    key: ENG-123
    title: "Add user authentication"
    type: story
    acceptance_criteria:
      - "User can log in with email and password"
      - "Session expires after 24 hours"
      - "Failed login shows error message without leaking details"

  design_context:                 # From Figma / Zeplin (UI changes only)
    matched_components: []
    specs: []

  rag_patterns:                   # From AWS Bedrock / GitHub Graph RAG / etc.
    - pattern: "Auth store pattern"
      source: "Previous PR #88"
      content: "Use httpOnly cookies for token storage, not localStorage"

  url_sources:                    # From plain URL fetches
    - name: "Shared ESLint config"
      url: "https://raw.githubusercontent.com/org/standards/main/eslint.md"
      content: "..."

review_priorities:
  # Guide reviewers on what to focus on
  # ⚠️  If user_instructions.provided is true — reviewers MUST check findings against user_instructions first.
  critical:
    - "Verify no v-html with user input (security requirement)"
    - "Check ESLint error-level rules compliance"
    - "Verify Pinia setup function style (ADR-002)"

  important:
    - "Check component naming (multi-word required)"
    - "Verify props have types and defaults"
    - "Check Container/Presentational pattern adherence"

  low_priority:
    - "Minor style preferences"
    - "Optional optimizations"

reviewer_guidance:
  # ⚠️  If user_instructions.provided is true:
  #     PREPEND each reviewer's list with user_instructions.focus_areas (if any)
  #     APPEND each reviewer's list with user_instructions.custom_requirements prefixed "MANDATORY: "
  #     Example: if focus_areas = ["JWT handling"], add "FOCUS: JWT handling" as first item
  general_review:
    - "Check for ESLint rule violations (no-var, prefer-const)"
    - "Verify component naming follows standards"
    - "Check inline annotations are followed"

  security_review:
    - "Flag ANY v-html usage (critical)"
    - "Check input validation per inline annotations"
    - "Verify no hardcoded secrets"

  performance_review:
    - "Check computed vs methods usage"
    - "Verify efficient filtering/searching"

  architecture_review:
    - "Verify Container/Presentational pattern"
    - "Check Pinia setup function style (not Options API)"
    - "Verify SRP adherence"

  business_review:
    - "Verify feature matches acceptance criteria"
    - "Check for user-facing regressions"
    - "Flag missing observability (logging, metrics) for new flows"

context_sources:
  # Track what sources were used
  primary_docs: [CLAUDE.md, AGENTS.md]
  config_files: [.eslintrc.js, .prettierrc]
  standards_docs: [CONTRIBUTING.md, ARCHITECTURE.md]
  inline_annotations: yes
  user_instructions: {true|false}  # true if user provided input in step-03
  mcp_tools: []                   # list of MCP tools actually used
  rag_systems: []                 # list of RAG systems queried
  url_sources: []                 # list of plain URLs fetched
```

### 3. Output Filename

Always: `pr-context.yaml` — the session folder already identifies the PR uniquely.

### 4. Write Knowledge Base to File

Write to: `{session_output}/pr-context.yaml`

Example: `_prr-output/reviews/2026-03-02-1430-pr123-feature-auth/pr-context.yaml`

### 5. Report Completion

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ PR-Specific Context Ready

📄 File: {filename}
📊 Stats:
   • ESLint rules: {n}
   • Guidelines: {m}
   • Inline annotations: {k}
   • User instructions: ⚠️ YES — scope: {scope}, focus: {focus_count} areas, requirements: {req_count}
     OR
   • User instructions: none — full standard review
   • MCP tools used: {mcp_list or "none"}
   • RAG patterns: {rag_count}
   • Issue context: {issue_key or "none"}

🎯 Domains: {domains}
📚 Sources: {source_count} ({list})

✓ Context is fresh and PR-specific
✓ Ready for review workflows
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 6. Store Knowledge Base Path in Working Context

Store `pr_knowledge_base` = `{session_output}/pr-context.yaml` in working context.

Review workflows (GR, SR, PR, AR, BR) find the knowledge base via working context or directly at `{session_output}/pr-context.yaml`.

### 7. Workflow Complete

Mark workflow as complete. Context is ready for use by review workflows (GR, SR, PR, AR, BR, IC).
