# Jira to GitHub AI Automation
<img width="1121" height="98" alt="image" src="https://github.com/user-attachments/assets/538ff9a7-ff39-467a-9e55-ec83dda0f94f" />

An n8n workflow that automatically fixes code bugs by reading Jira tickets 
and using Google Gemini AI to apply the fix directly to your GitHub repository.

## How It Works

1. **Jira ticket created** → workflow triggers automatically
2. **Fetches the file** from GitHub that needs to be fixed
3. **Sends the bug description + code** to Google Gemini AI
4. **AI generates the fixed code** and it gets validated
5. **Creates a new branch**, commits the fix, and opens a Pull Request
6. **Updates the Jira ticket** to reflect the change was made

## Setup

### Prerequisites
- n8n instance (cloud or self-hosted)
- Jira Software Cloud account
- GitHub account
- Google Gemini API key (free at [aistudio.google.com](https://aistudio.google.com))

### Configuration
1. Import the workflow JSON into n8n
2. Connect your **Jira** credentials
3. Connect your **GitHub** credentials  
4. Add your **Gemini API key** in the `Call Google Gemini` node
5. Update `YOUR_GITHUB_USERNAME` and `YOUR_REPO_NAME` in the relevant nodes
6. Activate the workflow

## Tech Stack
- **n8n** — workflow automation
- **Google Gemini Flash** — AI code fixing
- **GitHub API** — branch, commit, and PR creation
- **Jira API** — issue trigger and status update
