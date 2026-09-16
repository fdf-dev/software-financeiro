---
name: create-skill
description: "Use when: turning a recurring workflow, debugging process, review checklist, or implementation pattern into a reusable SKILL.md for this workspace or a personal workflow."
---

# Create Skill

Use this workflow when a repeatable process is emerging from a conversation, project work, or team practice and should become a reusable skill.

## Goal

Produce a clear `SKILL.md` that teaches when to use the workflow, the exact steps to follow, the decision points, and the completion criteria.

## Workflow

### 1. Extract the process from the conversation
Look for the actual methodology being used, such as:
- debugging approach
- code review checklist
- implementation pattern
- validation flow
- team operating procedure

Capture:
- the sequence of steps
- the order of operations
- the guardrails or constraints
- the expected outcome

### 2. Identify decision points and branching
Document where the workflow changes based on conditions, such as:
- missing information
- conflicting requirements
- failing verification
- project-specific constraints

Translate those conditions into simple rules like:
- if X is clear, do Y
- if X is missing, ask for clarification
- if verification fails, stop and diagnose root cause before patching

### 3. Define quality criteria
State what a successful outcome looks like. Include concrete completion checks, for example:
- the root cause was identified before fixing
- the change was validated with the relevant check
- the final result is consistent with the user's stated goal
- the workflow is clear enough to be reused without additional debate

### 4. Decide scope
Choose the right scope before finalizing:
- Workspace-scoped: use for team or project-specific operations, stored under `.github/skills/<name>/SKILL.md`
- Personal-scoped: use for cross-workspace habits or private workflows, stored in the user-level prompt directory

### 5. Draft the skill
Create a `SKILL.md` with a compact structure:
- frontmatter with `name` and meaningful `description`
- a short purpose or goal
- the step-by-step workflow
- decision branches and checks
- quality bar or completion criteria
- examples of when to use it

### 6. Refine the ambiguous parts
After the first draft, review weak areas and ask about them if needed:
- Is the outcome clear?
- Are the instructions specific enough to repeat?
- Are decision points explicit enough to guide action?
- Is the completion checklist measurable?

### 7. Save and validate
Confirm that:
- the file is in the correct location
- the YAML frontmatter is valid
- the description clearly indicates when the skill is useful
- the workflow is practical and reusable

## Completion checks

A good skill is ready when:
- it describes a real, repeatable workflow
- the steps are ordered and actionable
- decision branches are concrete and easy to follow
- the success criteria are testable
- the skill can be reused without needing the original conversation

## Example prompts

- "Turn this debugging workflow into a reusable skill."
- "Package our review checklist into a SKILL.md for the repo."
- "Create a reusable skill for handling feature implementation and validation."
- "Generalize this workflow into a personal skill for cross-project work."

## Related customizations

- create-instructions for always-on guidance
- create-prompt for single-purpose tasks
- create-agent for context-isolated multi-step workflows
- hooks for deterministic enforcement of rules
