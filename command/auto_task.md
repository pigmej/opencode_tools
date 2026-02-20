---
description: Create task based on the discussion from this session
---

**User Input:** $ARGUMENTS

Given the context of discussion, please create a new task in ./.task directory. To create task you MUST follow these steps.

# Preparations

1. new task id
You MUST find the file with biggest number in the ./.task directory. Add `10` to it. That's your {new-task-id}

2. short task title
You MUST use kebab-case maximum 7 words. This becomes {short-task-title}

3. file name
The resulting file name should be like `{new-task-id}-{short-task-title}.md` (replace variables with actual values from above), you MUST follow that naming schema.


# Task creation

## Task content
1. Make sure to include user feedback in the description
2. You MUST include Problem Statement in the description
3. You MUST include Requirements in the description
4. You MUST include Expected Outcome in the description
5. You MIGHT specify Additional suggestions and ideas depending on the User input which is: $ARGUMENTS
6. If during the discussion there were some important agreements about architecture decissions, about direction, about project context you MUST include them in the task also in the section "Other important agreements"

## Additional informations
1. You MIGHT want to read maximum a few files from ./.task directory to understand the format and User prefference. If you do so please try to follow format from the existing tasks



Give the user clear information about the created task file with created file path.
