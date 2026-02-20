---
description: "Review branch changes against base (default: main/master, or specify base)"
---

Review branch changes against the base branch for quality, security, and performance issues.

## Context

**Scope**: branch
**Base Branch**: !`if [ -n "$ARGUMENTS" ]; then echo "$ARGUMENTS"; elif git rev-parse --verify main >/dev/null 2>&1; then echo "main"; elif git rev-parse --verify master >/dev/null 2>&1; then echo "master"; else echo "main"; fi`
**Description**: All commits on current branch since diverging from base

## Diff to Review

!`mkdir -p .tmp && DIFF_FILE=".tmp/branch-diff-$(date +%Y%m%d-%H%M%S).txt" && BASE="${ARGUMENTS:-$(git rev-parse --verify main >/dev/null 2>&1 && echo main || echo master)}" && git diff $(git merge-base HEAD "$BASE")...HEAD > "$DIFF_FILE" && echo "Diff saved to: $DIFF_FILE ($(wc -l < "$DIFF_FILE" | tr -d ' ') lines)"`

Use the Read tool to review the diff file shown above.

## Instructions

### Step 1: Validate Input

If the diff above is empty, respond with "No branch changes to review (branch is up to date with base)." and stop.

### Step 2: Spawn Parallel Reviews (CRITICAL)

You MUST spawn all THREE subagents in a SINGLE Task tool batch for parallel execution:

1. **@code-review**: Pass the diff file path and instruct it to read and analyze for code quality issues
2. **@security-review**: Pass the diff file path and instruct it to read and analyze for security vulnerabilities
3. **@perf-review**: Pass the diff file path and instruct it to read and analyze for performance issues

Each subagent should return JSON with a `comments` array.

### Step 3: Merge Results

After all subagents complete:

1. Parse JSON from each subagent response
2. Add category field to each comment:
   - @code-review → `"category": "code"`
   - @security-review → `"category": "security"`
   - @perf-review → `"category": "performance"`
3. Combine all comments into a single array
4. If a subagent failed, note it in the summary but continue with other results

### Step 4: Display Summary

Output a formatted summary:

```
## Code Review Summary

**Base branch**: {base}

### Security ({count} issues)
- **{severity}** `{path}:{line}` - {message}

### Code Quality ({count} issues)  
- **{severity}** `{path}:{line}` - {message}

### Performance ({count} issues)
- **{severity}** `{path}:{line}` - {message}

---
**Total**: {n} issues ({x} errors, {y} warnings, {z} info/suggestions)
```

### Step 5: Save Report

1. Create `.reviews/` directory: `mkdir -p .reviews`
2. Save report to `.reviews/review-branch-{YYYY-MM-DDTHH-MM-SS}.md` with:
   - Base branch name
   - Summary (same as console output)
   - Full diff reviewed
   - Raw JSON with all comments
3. Report: `**Full review saved to**: .reviews/review-branch-{timestamp}.md`
