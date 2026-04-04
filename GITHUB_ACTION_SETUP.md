# GitHub Action Setup Guide

This guide will help you set up automatic Slack notifications for new Pull Requests using GitHub Actions.

## What is GitHub Actions?

GitHub Actions is a CI/CD platform that runs workflows directly in your GitHub repository. Unlike the polling script (`github-pr-notifier.py`), GitHub Actions:

- ✅ **Triggers instantly** when a PR is created (no 5-minute delay)
- ✅ **No server needed** - runs on GitHub's infrastructure
- ✅ **No rate limits** - event-driven, not polling
- ✅ **More reliable** - built-in retry logic
- ✅ **Free for public repos** - included in GitHub

## Prerequisites

1. A GitHub repository where you want to monitor PRs
2. Admin access to the repository (to add secrets)
3. A Slack webhook URL (see below if you don't have one)

## Step-by-Step Setup

### Step 1: Get Your Slack Webhook URL

If you already have a webhook URL from the `.env` file, you can use that. Otherwise:

1. Go to https://api.slack.com/messaging/webhooks
2. Click **"Create New App"** → **"From scratch"**
3. Name it "GitHub PR Notifier" and select your workspace
4. Click **"Incoming Webhooks"** → Toggle **ON**
5. Click **"Add New Webhook to Workspace"**
6. Select your channel → Click **"Allow"**
7. **Copy the Webhook URL** (starts with `https://hooks.slack.com/services/...`)

### Step 2: Add Webhook URL to GitHub Secrets

1. Go to your GitHub repository
2. Click **Settings** → **Secrets and variables** → **Actions**
3. Click **"New repository secret"**
4. Name: `SLACK_WEBHOOK_URL`
5. Value: Paste your Slack webhook URL
6. Click **"Add secret"**

### Step 3: Add the Workflow File to Your Repository

The workflow file has already been created at `.github/workflows/pr-slack-notification.yml`.

You need to commit and push it to your GitHub repository:

```bash
# Add the workflow file
git add .github/workflows/pr-slack-notification.yml

# Commit the changes
git commit -m "Add GitHub Action for Slack PR notifications"

# Push to your repository
git push origin main
```

**Note:** Replace `main` with your default branch name if different (e.g., `master`).

### Step 4: Test the Workflow

1. Create a test Pull Request in your repository
2. The GitHub Action should trigger automatically
3. Check your Slack channel for the notification
4. You can also view the workflow run:
   - Go to your repository → **Actions** tab
   - Click on the latest workflow run to see logs

## Workflow Features

The GitHub Action sends rich Slack notifications with:

- 🔔 **Header**: "New Pull Request"
- 📦 **Repository name**
- 🔢 **PR number**
- 📝 **PR title**
- 👤 **Author username**
- 🌿 **Branch names** (source → target)
- ✅ **Status** (Draft or Ready for Review)
- 📄 **PR description**
- 🔗 **Buttons**: "View Pull Request" and "View Files Changed"

## Customization

### Trigger on Different Events

Edit `.github/workflows/pr-slack-notification.yml` to change when notifications are sent:

```yaml
on:
  pull_request:
    types: 
      - opened          # New PR created
      - reopened        # Closed PR reopened
      - ready_for_review # Draft PR marked ready
      - review_requested # Reviewer assigned
```

### Customize the Message

You can modify the `payload` section in the workflow file to change:
- Message text and formatting
- Which PR details to include
- Button labels and actions
- Colors and emojis

### Filter by Branch

Only notify for PRs to specific branches:

```yaml
on:
  pull_request:
    types: [opened, reopened]
    branches:
      - main
      - develop
```

## Troubleshooting

### Notifications not appearing

1. **Check GitHub Actions tab**:
   - Go to your repository → Actions
   - Look for failed workflow runs
   - Click on a run to see error logs

2. **Verify the secret**:
   - Settings → Secrets and variables → Actions
   - Make sure `SLACK_WEBHOOK_URL` exists and is correct

3. **Test the webhook manually**:
   ```bash
   curl -X POST -H 'Content-type: application/json' \
   --data '{"text":"Test from GitHub Actions"}' \
   YOUR_WEBHOOK_URL
   ```

4. **Check workflow file syntax**:
   - GitHub will show a warning if YAML syntax is invalid
   - Use a YAML validator if needed

### Workflow not triggering

1. Make sure the workflow file is in the correct location: `.github/workflows/`
2. Verify the file is committed and pushed to the repository
3. Check that you have the correct branch name in your push command
4. Ensure the repository has Actions enabled (Settings → Actions → General)

### 400 Bad Request errors

This usually means the Slack webhook URL format is incorrect or the payload structure doesn't match what Slack expects. The workflow file provided uses the correct format for Slack's Block Kit API.

## Comparison: GitHub Actions vs Polling Script

| Feature | GitHub Actions | Polling Script |
|---------|---------------|----------------|
| **Trigger Speed** | Instant | 5 minutes |
| **Infrastructure** | GitHub's servers | Your server/computer |
| **Rate Limits** | None | GitHub API limits |
| **Reliability** | High (built-in retries) | Depends on your setup |
| **Cost** | Free (public repos) | Server/electricity costs |
| **Maintenance** | None | Keep script running |
| **Setup** | One-time | Continuous monitoring |

## Migration from Polling Script

If you're currently using the `github-pr-notifier.py` script:

1. Set up the GitHub Action (follow steps above)
2. Test that it works by creating a test PR
3. Once confirmed working, stop the polling script:
   ```bash
   # Find the process
   ps aux | grep github-pr-notifier.py
   
   # Kill it
   kill [PID]
   ```
4. You can keep the script as a backup or for local testing

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Slack Block Kit Builder](https://app.slack.com/block-kit-builder) - Design custom messages
- [Slack API Documentation](https://api.slack.com/messaging/webhooks)

## Support

If you encounter issues:
1. Check the GitHub Actions logs in your repository
2. Verify your Slack webhook URL is correct
3. Test the webhook manually with curl
4. Check that the secret is properly set in GitHub

---

**Note**: This GitHub Action approach is recommended over the polling script for production use due to its reliability, instant notifications, and zero maintenance requirements.