---
name: create-github-project-issue
description: >-
  Creates an issue on a GitHub project using the GitHub CLI. Use when the user
  asks to create, open, or file a GitHub issue on a project board, or
  reports a bug, defect, problem, enhancement, story, or task related to a project.
---

# Create GitHub Project Issue

## Overview
- Uses GitHub CLI (`gh`) to create an issue on a GitHub project board.
- Two issue types: **Bug** and **Enhancement**.
- Requires user approval before executing `gh project item-create`.

## When to Use
- Use this skill when the user requests to create a new issue on a GitHub project board, or reports a bug, defect, problem, enhancement, story, or task related to a project.
- Do not use for editing, closing, or deleting existing project items, adding comments, managing assignees/reviewers/milestones, creating pull requests, or non-GitHub project boards.

## Instructions
1. **Classify** - Bug or Enhancement
1. **Resolve** - Determine Project Board (See `Notes -> Project Resolution` section)
1. **Preflight** - Run preflight checks; on failure, explain and wait for further instructions.
1. **Gather** - Collect required fields; stop and wait until complete.
1. **Draft** - Show draft of issue to be created, including all gathered fields and labels; ask for confirmation to proceed (see `Output Format -> Draft` section).
1. **Create** - On approval, run `gh project item-create` with appropriate flags.
1. **Output** - Show link to created issue and any relevant next steps (see `Output Format -> Success Message` section).

### Classify
Unless otherwise determined by the prompt, classify the issue as either a **Bug** or **Enhancement** using the below table. If unclear, ask the user.
| Type            | Use when                                               | Required Label      |
|-----------------|--------------------------------------------------------|---------------------|
| **Bug**         | Defect, problem, bug, broken behavior, regession error | `type: bug`         |
| **Enhancement** | Everything else: enhancement, feature, story, task     | `type: enhancement` |

### Resolve
Determine the GitHub project board for which to create the issue.

- If the user specifies a project, use that.
- Else if the user only has one project, use that.
- Else ask the user to choose from their projects.

### Preflight
Run after project resolution is complete. Always run again immediately before any `gh` command if time has passed or context changed.

- **Auth** `gh auth status`; on failure, show error, suggest `gh auth login`, stop and wait.
- **Project Access** `gh project view PROJECT_NUMBER --owner @me`; on failure, show error, confirm project and permissions, stop and wait.

### Gather
Depending on the type of issue being created, gather the following information from the user.

- **Title** (required): A one-line summary of the issue.
- **Summary** (required): A brief description of the issue, including any relevant context, affected users, and expected impact.
- **Steps to Reproduce** (required for Bugs): A numbered list of steps to reproduce the issue.
- **Expected Result** (required for Bugs): A description of the behavior expected by the user.
- **Actual Result** (required for Bugs): A description of the actual behavior experienced by the user.
- **Acceptance Criteria** (required for Enhancements): A bulleted list of changes expected to be present after work is complete on the issue.
- **Additional Information** (optional): Any other relevant information, including by not limited to: screenshots, error messages, links to related issues.

Information for any of these fields can be determined from the context of the user's prompt, but if any required information is missing or unclear, ask targeted questions to gather it and **stop** until answered. Do not draft until all required information is known.

### Draft
Utilizing the appropriate template, build the body of the issue to be created. 

#### Bug Template
``````markdown
## Summary
...

## Steps to Reproduce
1. ...
1. ...
1. ...
...

## Expected Result
...

## Actual Result
...

## Additional Information
...
``````

#### Enhancement Template
``````markdown
## Summary
...

## Acceptance Criteria
- ...
- ...
- ...
...

## Additional Information
...
``````

Present the following in a single message to the user, asking for confirmation to proceed. Stop and wait for confirmation before creating issue.
```markdown
- **Type**: Bug | Enhancement
- **Project**: Project Name
- **Title**: Issue Title
- **Labels**: type: bug | type: enhancement, plus any additional labels the user requested
- **Body**: full markdown preview
```

As the user to approve, request edits, or cancel.
| User Response | User Might Say                        | Action                                             |
|---------------|---------------------------------------|----------------------------------------------------|
| Approve       | "Looks good", "Create it", "Approved" | Run `gh project item-create`                       |
| Request Edits | "Please change X", "Add more to Y"    | Update draft, present again, **stop** for approval |
| Cancel        | "Cancel", "Never mind", "Stop"        | Do not create issue, end flow; do not run `gh`     |

### Create
The agent will use the GitHub CLI to create the issue on the resolved project board.
```bash
gh project item-create PROJECT_NUMBER \
  --owner @me \
  --title "Issue Title" \
  --label "type: bug" | "type: enhancement" \
  --label "additional label" \ # if needed
  --body "full markdown body" 
```

- Always include the label "type: bug" or "type: enhancement" based on issue type.
- Capture stdout and std error for outcome reporting.

### Output
- **Success**: Report exactly `Issue created successfully: ISSUE_URL` using the URL from `gh` stdout. Do not invent a URL.
- **Failure**: Report exactly `Failed to create issue: ERROR_MESSAGE` using the error message from `gh` stderr. Do not claim success.

