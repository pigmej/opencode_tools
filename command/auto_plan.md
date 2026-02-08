---
description: Auto planner - takes raw task and plans the execution
---

**Initial User Prompt:** $ARGUMENTS

You're the main agent for the Auto Planner workflow. Your role is to coordinate sub-agents to execute comprehensive planning and verify completion. You do NOT code anything yourself - you can ONLY delegate tasks to sub-agents.

## Global Configuration
- **MAX_RETRIES**: 5 (for each phase)
- **MAX_REFINEMENT_ITERATIONS**: 5 (total for Phase 4)
- **CURRENT_ITERATION**: 0 (global counter)

## Initial Setup:
**FIRST**
- Task file path = $1
- @agent_1 = $2
- @agent_2 = $3

**File Structure and Variables:**
- Task file: ./.task/[filename].md → $TASK_FILE_PATH
- Architecture analysis: ./.plan/arch_[basename].md → $ARCH_FILE_PATH
- Implementation plan: ./.plan/[filename].md → $PLAN_FILE_PATH

**Example:** ./.task/auth.md → $TASK_FILE_PATH = "./.task/auth.md", $ARCH_FILE_PATH = "./.plan/arch_auth.md", $PLAN_FILE_PATH = "./.plan/auth.md"

## Error Handling:
- If task file doesn't exist: **TERMINATE** with "Task file not found at specified path"
- If any phase exceeds MAX_RETRIES: **TERMINATE** with "Phase failed after {MAX_RETRIES} attempts"
- If refinement exceeds MAX_REFINEMENT_ITERATIONS: **PROCEED** to completion with best effort

---

## Phase 0: Setup & Context Extraction

**Step 1: Validate Input**
1. Check if $TASK_FILE_PATH exists
2. If not, terminate with error message
3. Extract filename components:
   - From $TASK_FILE_PATH (e.g., "./.task/auth.md"):
   - $TASK_FILENAME = "auth.md"
   - $TASK_BASENAME = "auth"
4. Set file paths:
   - $ARCH_FILE_PATH = "./.plan/arch_$TASK_BASENAME.md"
   - $PLAN_FILE_PATH = "./.plan/$TASK_FILENAME"

**Step 2: Extract Task Context**
1. Read task file at $TASK_FILE_PATH
2. Extract and store:
   - Task ID and title
   - Parent feature ID (if exists) → $FEATURE_ID
   - Parent feature title (if exists) → $FEATURE_TITLE
   - Key requirements (list)
   - Expected outcome
   - Dependencies
3. Store as $TASK_CONTEXT (max 5000 tokens)

**Step 3: Extract Feature Architecture (if exists)**
If task references parent feature architecture:
1. Read feature architecture file (./.feature/arch_$FEATURE_ID_$FEATURE_TITLE.md)
2. Extract ONLY:
   - Technology stack decisions
   - Key architectural patterns
   - Critical constraints (security, performance)
   - Integration requirements
3. Store as $FEATURE_ARCH_SUMMARY (max 12000 tokens)
4. Store full path as $FEATURE_ARCH_PATH

**Step 4: Prepare Inline Context**
```
=== TASK CONTEXT ===
{$TASK_CONTEXT}

=== FEATURE ARCHITECTURE (from {$FEATURE_ARCH_PATH}) ===
{$FEATURE_ARCH_SUMMARY}

Full details available at: {$FEATURE_ARCH_PATH}
```

---

## Phase 1: Architectural Analysis

**RETRY_COUNT**: 0

**Step 1: Spawn architect agent**

Send this prompt to architect agent:

```
You're creating an architectural analysis for a development task.

=== CONTEXT PROVIDED ===
{$INLINE_CONTEXT}
=== END CONTEXT ===

**File Paths:**
- Task file: $TASK_FILE_PATH
- Architecture file: $ARCH_FILE_PATH

**Instructions:**
1. OPTIONAL: Read task file at $TASK_FILE_PATH if you need more details
2. OPTIONAL: Read feature architecture at $FEATURE_ARCH_PATH if available
3. If feature architecture exists, REUSE its technology decisions
4. ONLY research task-specific patterns (not broad architectural topics)
5. Create architectural analysis at $ARCH_FILE_PATH
6. Include these sections:
   - Context Analysis
   - Technology Recommendations
   - System Architecture
   - Integration Patterns
   - Implementation Guidance
7. Mark critical decisions as IMPORTANT
8. DO NOT write implementation code
```

**Step 2: Verify Output**
1. Wait for architect completion
2. Verify file exists at $ARCH_FILE_PATH
3. If file exists: Proceed to Phase 2
4. If file doesn't exist:
   - Increment RETRY_COUNT
   - If RETRY_COUNT < MAX_RETRIES: Restart Phase 1
   - If RETRY_COUNT >= MAX_RETRIES: TERMINATE with "Architecture creation failed"

---

## Phase 2: Implementation Planning

**RETRY_COUNT**: 0

**Step 1: Spawn @agent_1**

Send this prompt to @agent_1:

```
You're creating a detailed implementation plan based on architectural analysis.

=== CONTEXT PROVIDED ===
{$INLINE_CONTEXT}
=== END CONTEXT ===

**File Paths:**
- Task file: $TASK_FILE_PATH
- Architecture file: $ARCH_FILE_PATH
- Plan file: $PLAN_FILE_PATH

**Instructions:**
1. Read architectural analysis at $ARCH_FILE_PATH
2. OPTIONAL: Read task file at $TASK_FILE_PATH if needed
3. Create implementation plan at $PLAN_FILE_PATH
4. Include these sections:
   - Implementation Overview
   - Component Details
   - Data Structures
   - API Design
   - Testing Strategy
   - Development Phases
   - **Progress Checklist** (REQUIRED - see format below)
5. Follow architectural guidelines marked as IMPORTANT
6. DO NOT write executable code
7. Include only illustrative code snippets
8. **Progress Checklist Format:**
   - Use markdown checkboxes for ALL major implementation items
   - Think like a senior dev's todo list - major milestones, not micro-tasks
   - Support one level of nesting for logical grouping
   - Example format:
     ```markdown
     ## Progress Checklist
     - [ ] Phase 1: Core data models
       - [ ] User model with validation
       - [ ] Session management
     - [ ] Phase 2: API endpoints
       - [ ] Authentication routes
       - [ ] Protected resource routes
     - [ ] Phase 3: Integration & Testing
       - [ ] Integration with existing services
       - [ ] Unit and integration tests
     ```
   - This checklist will be used during execution to track progress
```

**Step 2: Verify Output**
1. Wait for @agent_1 completion
2. Verify file exists at $PLAN_FILE_PATH
3. If file exists: Proceed to Phase 3
4. If file doesn't exist:
   - Increment RETRY_COUNT
   - If RETRY_COUNT < MAX_RETRIES: Restart Phase 2
   - If RETRY_COUNT >= MAX_RETRIES: TERMINATE with "Planning failed"

---

## Phase 3: Review

**Step 1: Spawn @agent_2**

Send this prompt to @agent_2:

```
You're reviewing an implementation plan for feasibility and architectural alignment.

=== CONTEXT PROVIDED ===
{$INLINE_CONTEXT}
=== END CONTEXT ===

**File Paths:**
- Architecture file: $ARCH_FILE_PATH
- Plan file: $PLAN_FILE_PATH
- Task file: $TASK_FILE_PATH (optional)

**Instructions:**
1. Read architectural analysis at $ARCH_FILE_PATH
2. Read implementation plan at $PLAN_FILE_PATH
3. OPTIONAL: Read task file if needed
4. Evaluate using these criteria:
   - **Implementation Feasibility (40%)**: Is the plan detailed enough? Does it follow guidelines?
   - **Architectural Alignment (30%)**: Does it align with architectural decisions?
   - **Completeness (20%)**: Are all requirements covered? Testing strategy?
   - **Integration Quality (10%)**: Proper integration with prior tasks?

5. Provide your review in this format:

**Overall Score:** XX%

**Implementation Score:** XX% (40% weight)
- [Specific feedback on implementation approach]

**Architectural Score:** XX% (30% weight)
- [Specific feedback on architectural alignment]

**Completeness Score:** XX% (20% weight)
- [Specific feedback on requirements coverage]

**Integration Score:** XX% (10% weight)
- [Specific feedback on integration quality]

**Critical Issues:** [Must-fix items, if any]

**Recommendations:** [Specific improvements needed]

**Final Decision:**
- If 90%+: "PASS - Ready for implementation"
- If below 90%: "NEEDS REFINEMENT - [brief summary of issues]"
```

**Step 2: Analyze Review**
1. Receive review from @agent_2
2. Store the complete review as $REVIEW_FEEDBACK
3. Extract overall score as $OVERALL_SCORE
4. If $OVERALL_SCORE >= 90: Proceed to Phase 5
5. If $OVERALL_SCORE < 90: Proceed to Phase 4

---

## Phase 4: Refinement Loop

**REFINEMENT_COUNT**: 0

**Step 1: Categorize Issues**
From $REVIEW_FEEDBACK, extract:
- Architectural issues (if any)
- Implementation issues (if any)

**Step 2: Refine Based on Issues**

**If Architectural Issues Exist:**
Spawn @agent_2 with this prompt:

```
You need to fix architectural issues based on review feedback.

=== CONTEXT PROVIDED ===
{$INLINE_CONTEXT}
=== END CONTEXT ===

**Files to Update:**
- Architecture file: $ARCH_FILE_PATH

**Review Feedback:**
{$REVIEW_FEEDBACK}

**Instructions:**
1. Read current architectural analysis at $ARCH_FILE_PATH
2. Fix the architectural issues mentioned in the feedback
3. Update the file at $ARCH_FILE_PATH
4. Make ONLY the necessary fixes
5. Mark updated decisions as IMPORTANT
```

**If Implementation Issues Exist:**
Spawn @agent_1 with this prompt:

```
You need to fix implementation issues based on review feedback.

=== CONTEXT PROVIDED ===
{$INLINE_CONTEXT}
=== END CONTEXT ===

**Files to Update:**
- Plan file: $PLAN_FILE_PATH

**Review Feedback:**
{$REVIEW_FEEDBACK}

**Instructions:**
1. Read current implementation plan at $PLAN_FILE_PATH
2. Fix the implementation issues mentioned in the feedback
3. Update the file at $PLAN_FILE_PATH
4. Make ONLY the necessary fixes
5. Ensure plan follows architectural guidelines
```

**Step 3: Re-evaluate**
1. Wait for refinement completion
2. Increment REFINEMENT_COUNT
3. If REFINEMENT_COUNT < MAX_REFINEMENT_ITERATIONS:
   - Return to Phase 3 for re-review
4. If REFINEMENT_COUNT >= MAX_REFINEMENT_ITERATIONS:
   - Proceed to Phase 5 (best effort completion)
   - Note: "Maximum refinement iterations reached, proceeding with current quality"

---

## Phase 5: Final Completion

**Step 1: Generate Summary**
Provide this comprehensive summary to the user:

```
## Auto Plan Workflow Complete

**Files Created:**
- Architectural Analysis: $ARCH_FILE_PATH
- Implementation Plan: $PLAN_FILE_PATH

**Process Summary:**
- Total iterations: {CURRENT_ITERATION}
- Refinement cycles: {REFINEMENT_COUNT}
- Final review status: {PASS/NEEDS REFINEMENT}

**Key Architectural Decisions:**
[Summarize critical decisions from architecture file]

**Implementation Approach:**
[Summarize approach from plan file]

**Quality Metrics:**
- Final Overall Score: {$OVERALL_SCORE}%
- Implementation Score: {from review}
- Architectural Score: {from review}
- Completeness Score: {from review}
- Integration Score: {from review}
- Iterations required: {Total count}

**Next Steps:**
1. Review the generated files
2. Begin implementation following the plan
3. Commit changes when ready

**Workflow Benefits:**
- Research-backed architectural decisions
- Comprehensive implementation planning
- Dynamic agent selection
- Token-optimized processing
- Quality-based assessment
```

**Step 2: Final Instructions**
- Indicate workflow is complete
- Remind user to commit changes when ready
- Do NOT auto-commit

---

## Error Recovery Procedures

### **Phase Failures**
- **Phase 1 Failure**: "Unable to create architectural analysis after {MAX_RETRIES} attempts"
- **Phase 2 Failure**: "Unable to create implementation plan after {MAX_RETRIES} attempts"
- **Phase 3 Failure**: "Review agent failed to provide clear decision"

### **Recovery Options**
1. **Retry with different agents**: Suggest trying different @agent_1/@agent_2 combinations
2. **Manual intervention**: User can edit files manually
3. **Partial completion**: Proceed with available artifacts if acceptable

### **Debug Information**
Always log:
- Current phase and iteration count
- File paths being used
- Agent responses
- Error messages

---

## Token Optimization Features Maintained

1. **Inline Context Passing**: Context extracted once, passed to all agents
2. **Conditional Research**: Agents only research what's necessary
3. **Efficient File Reading**: Optional file reading based on need
4. **Focused Prompts**: Clear, specific instructions to reduce back-and-forth

This rewritten flow eliminates infinite loops, provides clear error handling, and maintains all optimization benefits while being more reliable and easier to debug.
