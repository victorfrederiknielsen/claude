---
name: create-pr
description: Create a GitHub PR for the current branch using gh CLI. Automatically generates a professional PR description using the pr-description skill. Use when asked to create a PR, open a PR, or submit changes for review.
---

# Create GitHub Pull Request

Create a pull request for the current branch with a professionally written description.

## Prerequisites

- `gh` CLI must be installed and authenticated (`gh auth status`)
- Current branch must have commits not yet on the target branch
- Remote must be configured

## Workflow

### Step 1: Verify Environment

```bash
# Check gh is authenticated
gh auth status

# Get current branch
git branch --show-current

# Verify there are changes to push
git log origin/master..HEAD --oneline
```

### Step 2: Extract Ticket Number

Extract the ticket number to prefix the PR title. Check these sources in order:

#### From Branch Name

Branch patterns to match (case-insensitive, will be uppercased):
- `user/XXX-123-description` → `XXX-123`
- `XXX-123-description` → `XXX-123`
- `user/ram-27-some-feature` → `RAM-27`

```bash
# Get branch name and extract ticket
BRANCH=$(git branch --show-current)

# Extract ticket (handles both user/XXX-123-desc and XXX-123-desc formats)
# Use -i for case-insensitive match, then uppercase the result
TICKET=$(echo "$BRANCH" | grep -oiE '[A-Za-z]+-[0-9]+' | head -1 | tr '[:lower:]' '[:upper:]')

echo "Ticket: $TICKET"
```

#### From Commit Messages (fallback)

If no ticket in branch name, check commit messages:

```bash
# Look for ticket pattern in commit messages (case-insensitive, then uppercase)
git log origin/master..HEAD --format="%s" | grep -oiE '[A-Za-z]+-[0-9]+' | head -1 | tr '[:lower:]' '[:upper:]'
```

#### Ticket Number Rules

- Pattern: `XXX-NN` where `XXX` is 2-5 uppercase letters and `NN` is 1+ digits
- Examples: `RAM-27`, `RAA-456`, `JIRA-1234`
- If no ticket found, create PR without prefix (but warn user)

### Step 3: Generate PR Description

**IMPORTANT:** Use the `pr-description` skill to generate the description.

Invoke the skill:
```
skill: "pr-description"
```

Follow the pr-description skill's workflow to analyze changes and generate a proper description with:
- Summary (imperative mood)
- Changes section
- Testing Strategy section

### Step 4: Push Branch (if needed)

```bash
# Check if branch exists on remote
git ls-remote --heads origin $(git branch --show-current)

# If not pushed yet, push with upstream tracking
git push -u origin $(git branch --show-current)
```

### Step 5: Create the PR

Use `gh pr create` with a heredoc to preserve formatting.

**IMPORTANT:** Prefix the title with the ticket number extracted in Step 2.

Title format: `TICKET-123 Description in imperative mood`

```bash
# With ticket number
gh pr create --title "RAM-27 Add user profile settings" --body "$(cat <<'EOF'
## Summary
[Generated summary from pr-description skill]

## Changes
[Generated changes from pr-description skill]

## Testing Strategy
[Generated testing strategy from pr-description skill]
EOF
)"

# Without ticket (if none found - warn user)
gh pr create --title "Add user profile settings" --body "..."
```

### Step 6: Confirm and Share

After creating the PR:
1. Display the PR URL to the user
2. Optionally open in browser with `gh pr view --web`

## Options

The user may specify:
- **Target branch**: Default is `master` or `main`, but user can specify another (e.g., `develop`)
- **Draft**: Create as draft PR with `--draft` flag
- **Reviewers**: Add reviewers with `--reviewer @username`
- **Labels**: Add labels with `--label bug,urgent`

### Example with Options

```bash
gh pr create \
  --title "Fix authentication timeout" \
  --body "$(cat <<'EOF'
## Summary
...
EOF
)" \
  --base develop \
  --draft \
  --reviewer @teammate \
  --label bug
```

## Error Handling

### Branch Not Pushed
```bash
# Error: branch not found on remote
git push -u origin $(git branch --show-current)
```

### No Commits to Compare
```bash
# Check if there are actual differences
git diff origin/master...HEAD --stat
```

### PR Already Exists
```bash
# Check for existing PR
gh pr list --head $(git branch --show-current)

# If exists, show it instead
gh pr view
```

## Complete Example Flow

1. **User says:** "Create a PR for my changes"

2. **Check environment:**
   ```bash
   gh auth status
   git branch --show-current
   git log origin/master..HEAD --oneline
   ```

3. **Extract ticket number:**
   ```bash
   # Branch: vic/ram-27-add-profile-settings (or vic/RAM-27-add-profile-settings)
   BRANCH=$(git branch --show-current)
   TICKET=$(echo "$BRANCH" | grep -oiE '[A-Za-z]+-[0-9]+' | head -1 | tr '[:lower:]' '[:upper:]')
   # Result: RAM-27 (always uppercase)
   ```

4. **Invoke pr-description skill** to analyze changes and generate description

5. **Push branch if needed:**
   ```bash
   git push -u origin $(git branch --show-current)
   ```

6. **Create the PR with ticket prefix:**
   ```bash
   gh pr create --title "RAM-27 Add user profile settings page" --body "$(cat <<'EOF'
   ## Summary
   Add user profile settings page allowing users to update their display name, email preferences, and notification settings.

   ## Changes
   - Add ProfileSettings component with form validation
   - Create useProfileSettings hook for state management
   - Add API endpoints for fetching and updating profile data
   - Include comprehensive test coverage

   ## Testing Strategy
   - [ ] Verify form validation shows appropriate errors
   - [ ] Test successful profile update saves to backend
   - [ ] Check notification preferences toggle works
   - [ ] Verify changes persist after page reload
   EOF
   )"
   ```

7. **Share the result:**
   ```
   PR created: https://github.com/org/repo/pull/123
   ```

## Tips

- Always prefix PR title with ticket number (e.g., `RAM-27 Add feature`) for traceability
- Always let the pr-description skill generate the description for consistency
- Use `--draft` when changes aren't ready for final review
- Add relevant labels to help with PR organization
- Tag appropriate reviewers if you know who should review
- If no ticket found in branch or commits, warn the user and ask if they want to add one manually
