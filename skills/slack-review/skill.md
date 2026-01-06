---
name: slack-review
description: Generate a Slack-formatted code review request post with PR and Linear ticket links. Use after creating a PR or when asked to generate a Slack review post.
---

# Slack Code Review Post Generator

Generate a nicely formatted Slack post for requesting code reviews in your team channel.

## Output Format

Use markdown link syntax `[text](URL)` for clickable short labels. Description on first line, links inline on second line:

```
<Short one-liner description of the change>
:github: [REPO#PR_NUMBER](PR URL) :linearapp: [TICKET](Linear URL)
```

## Workflow

### Step 1: Get PR Information

If a PR URL was just created or is known, use it directly. Otherwise, fetch it:

```bash
# Get PR URL for current branch
gh pr view --json url -q '.url'

# Get PR title for the one-liner
gh pr view --json title -q '.title'
```

### Step 2: Extract Ticket Number

Get the ticket number from the branch name or PR title:

```bash
# Get branch name
BRANCH=$(git branch --show-current)

# Extract ticket (case-insensitive, then uppercase)
TICKET=$(echo "$BRANCH" | grep -oiE '[A-Za-z]+-[0-9]+' | head -1 | tr '[:lower:]' '[:upper:]')

echo "Ticket: $TICKET"
```

### Step 3: Build Linear URL

Construct the Linear ticket URL using the ticket number:

```
https://linear.app/instruqt/issue/<TICKET>
```

For example: `https://linear.app/instruqt/issue/RAM-27`

### Step 4: Generate One-Liner

Create a short, descriptive one-liner for the change. This should be:
- Brief (under 80 characters ideally)
- Descriptive of what the change does
- Written in sentence case

Sources for the one-liner (in order of preference):
1. PR title (remove ticket prefix if present)
2. First commit message summary
3. Ask user if unclear

### Step 5: Output the Slack Post

Copy the Slack post directly to clipboard using `pbcopy`:

```bash
echo -n 'Preserve scroll positions when switching markdown editor modes
:github: [frontend#123](https://github.com/instruqt/frontend/pull/123) :linearapp: [RAM-27](https://linear.app/instruqt/issue/RAM-27)' | pbcopy
```

Tell the user: "Slack post copied to clipboard! Just paste it in Slack."

## Example

**Input:** User just created PR #9759 for branch `vic/ram-27-previewing-instructions-markdown-doesnt-preserve-scrolling`

**Output:** Copied to clipboard:
```
Preserve scroll positions when switching markdown editor modes
:github: [frontend#9759](https://github.com/instruqt/frontend/pull/9759) :linearapp: [RAM-27](https://linear.app/instruqt/issue/RAM-27)
```

## Usage

This skill can be:
1. **Invoked manually:** User asks "generate slack review post" or "create slack post for PR"
2. **Called from create-pr skill:** After creating a PR, optionally generate the Slack post

## Tips

- Keep the one-liner concise and informative
- The Linear URL format is `https://linear.app/instruqt/issue/<TICKET>`
- If no ticket is found, omit the Linear line and warn the user
- Use `pbcopy` to copy directly to clipboard for easy pasting
