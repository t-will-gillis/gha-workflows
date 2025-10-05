# Add Update Label Weekly - Workflow Template

## Overview

This workflow automatically manages issue staleness labels based on assignee activity. It:
- Monitors issues in "In Progress (actively working)" status
- Adds labels when issues haven't been updated within specified timeframes
- Removes labels when issues are updated or have linked PRs
- Hides outdated bot comments

## When to Use This

Use this workflow if your project needs to:
- Track which issues need status updates from assignees
- Automatically label stale issues
- Maintain visibility on inactive work
- Send reminders to assignees about updating their issues

## Prerequisites

### Required
1. **GitHub Token with elevated permissions**
   - Scope: `repo (full)` and `project (full)`
   - Needed because the workflow queries project board status fields via GraphQL
   - Store as a repository secret (e.g., `PROJECT_TOKEN`)

2. **Project-specific label configuration**
   - Your repo must have a label directory structure
   - Must implement `retrieve-label-directory.js` utility

3. **Project Board with specific status**
   - Must have a status field with "In progress (actively working)" option
   - Must implement `queryIssueInfo.js` to query project board data

### Optional
- Comment template file for customized update reminders
- Custom bot usernames if using organization-specific bots

## Installation

### 1. Copy Workflow File

Copy `add-update-label-weekly.yml` to your project repo:
```
project-repo/.github/workflows/add-update-label-weekly.yml
```

### 2. Set Up Project Structure

Create the following structure in your project repo:
```
project-repo/
├── .github/workflows/
│    └── add-update-label-weekly.yml
└── github-actions/
     ├── add-update-label/
     │    ├── add-label.js
     │    ├── config.js                    # Create this for customization
     │    └── update-instructions-template.md
     └── utils/
          ├── retrieve-label-directory.js
          ├── query-issue-info.js
          ├── find-linked-issue.js
          ├── get-timeline.js
          └── minimize-issue-comment.js
```

### 3. Customize the Workflow

Edit `add-update-label-weekly.yml`:

```yaml
# Adjust cron schedule to your project
schedule:
  - cron: '0 7 * 1-6,8-11 5'  # Change timing as needed

# Line 16: Update repository name
if: github.repository == 'YOUR_ORG/YOUR_REPO'

# Line 22: Use your token secret name
github-token: ${{ secrets.YOUR_TOKEN_SECRET }}
```

### 4. Create Configuration File

Create `github-actions/issue-staleness/config.js`:

```javascript
module.exports = {
  // Timeframes for determining issue staleness
  updatedByDays: 3,           // Issues updated within this are "current"
  commentByDays: 7,           // Issues not updated for this long get first warning
  inactiveUpdatedByDays: 14,  // Issues not updated for this long are "inactive"
  upperLimitDays: 35,         // Comments older than this get minimized
  
  // Project board configuration
  targetStatus: "In progress (actively working)",
  
  // Bot usernames that should have their comments minimized when outdated
  botUsernames: ["github-actions[bot]", "YourOrgBot"],
  
  // Labels to exclude from checking (issues with these labels are skipped)
  excludeLabels: ["Draft", "ER: ", "Epic", "Dependency", "Completed"],
};
```

### 5. Configure Repository Secret

1. Create a Personal Access Token (PAT) with `repo` and `project` scopes
2. Go to your repository Settings → Secrets and variables → Actions
3. Add a new repository secret with your chosen name (e.g., `MY_TOKEN`)
4. Use this name in the workflow file

## Customization Guide

### Adjust Staleness Thresholds

In `config.js`, modify the day values:
```javascript
updatedByDays: 5,           // More lenient: 5 days before warning
commentByDays: 10,          // First warning at 10 days
inactiveUpdatedByDays: 21,  // Inactive label at 21 days
```

### Change Schedule

In the workflow YAML:
```yaml
# Run at different time/day
- cron: '0 14 * * 1'  # Mondays at 2 PM UTC

# Run more frequently
- cron: '0 7 * * 1,5'  # Mondays and Fridays

# Run monthly
- cron: '0 7 1 * *'  # First day of each month
```

### Customize Comment Template

Edit `update-instructions-template.md` to change the message sent to assignees.

### Modify Which Issues Are Checked

In `add-label.js`, update `getIssueNumsFromRepo()` to:
- Check different status values
- Include/exclude different labels
- Filter by different criteria

## How It Works

### Issue Processing Logic

For each issue in "In Progress (actively working)" status:

1. **Check for linked PR by assignee**
   - If open PR exists that fixes/resolves/closes the issue → Remove all update labels

2. **Analyze timeline for last update**
   - Last assignee comment or assignment date is considered an "update"

3. **Apply labels based on time since update**:
   - **< 3 days**: Keep "Status: Updated" label (if present)
   - **3-7 days**: Remove all update-related labels
   - **7-14 days**: Add "To Update" label, post reminder comment
   - **> 14 days**: Add "Inactive" label, post stronger reminder

4. **Minimize outdated bot comments**
   - Comments older than 7 days (but newer than 35 days) are minimized

### Labels Used

The workflow expects these label keys in your label directory:
- `statusUpdated` - Issue was recently updated
- `statusInactive1` - Issue needs update (7-14 days)
- `statusInactive2` - Issue is inactive (14+ days)
- `draft`, `er`, `epic`, `dependency`, `skillsIssueCompleted` - Excluded from checks

## Troubleshooting

### Workflow doesn't run
- Check the `if: github.repository ==` condition matches your repo
- Verify the schedule syntax is correct
- Check that workflow is enabled in Actions tab

### Token permission errors
- Ensure PAT has `repo` and `project` scopes
- Verify secret name matches what's in the workflow
- Check token hasn't expired

### "Label not found" errors
- Verify your `retrieve-label-directory.js` returns the correct label names
- Check that labels actually exist in your repository

### GraphQL errors
- Ensure `queryIssueInfo.js` is properly configured
- Verify project board node IDs are correct
- Check that status field name matches your project board

### Issues not being processed
- Check console output for which issues are skipped and why
- Verify issue has the correct status in project board
- Check that issue doesn't have excluded labels

## Maintenance

### Regular Updates
- Review day thresholds quarterly to ensure they match team expectations
- Update bot usernames if you add new automation
- Adjust excluded labels as your label taxonomy evolves

### Monitoring
- Check workflow runs in the Actions tab
- Review comments posted to issues for appropriateness
- Gather feedback from team about label timing

## Support

- **Documentation**: See detailed migration plan in org documentation
- **Issues**: Report problems in the `.github` repository
- **Questions**: Contact DevOps team or ask in engineering Slack channel

## Related Workflows

This workflow pairs well with:
- Issue triage automation
- Project board automation
- PR review reminders

## Contributing

If you improve this workflow, consider:
1. Documenting your changes
2. Sharing back to the template
3. Helping other projects adopt improvements
