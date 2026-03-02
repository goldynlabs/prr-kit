---
title: "Performance Review Completion Checklist"
validation-target: "Performance review output file"
---

# Performance Review Checklist

## Coverage
- [ ] Database/query patterns checked (N+1, missing pagination, SELECT *)
- [ ] Async/await patterns reviewed
- [ ] Memory management reviewed
- [ ] Frontend performance checked (if frontend code changed)

## Finding Quality
- [ ] Every finding has: file path + line/function reference
- [ ] Every finding has: severity level (🔴/🟡/🟢/❓)
- [ ] Every finding has: estimated impact (high/medium/low) with brief rationale
- [ ] Micro-optimizations are NOT flagged (only impactful issues)
- [ ] Each finding includes suggested fix or question
- [ ] ❓ QUESTION findings include: specific concern + context needed to assess severity (e.g., "Is this in a hot path?", "What is the expected data volume?")

## Output
- [ ] Findings written to `{session_output}/performance-review.md`
