# Jira-GitHub-n8n_Autonomous-Agent
<img width="1728" height="123" alt="image" src="https://github.com/user-attachments/assets/14d5a18a-7feb-4d3f-9a76-fbad79428daf" />

## Overview
This project is a streamlined, autonomous n8n workflow that bridges issue tracking (Jira) and version control (GitHub). It acts as an AI-driven software engineer that automatically responds to newly created Jira bug tickets, fetches the relevant source code from your GitHub repository, writes Python code fixes via the Google Gemini API, pushes the revised commit, and opens a Pull Request (PR)—all without human intervention.

It is designed with strict guardrails to prevent AI hallucinations and ensure only valid source code is committed to your repository.

---

## Key Achievements & Breakthroughs
* **End-to-End Autonomous Resolution:** Transforms n8n into an automated pipeline capable of taking a bug from Jira ticket creation to a fully staged GitHub Pull Request in seconds.
* **Strict Output Enforcement:** Bypasses typical LLM conversational behaviors. The system explicitly instructs Gemini to output only raw code, and programmatically strips out markdown fences, headers, or conversational prose.
* **Syntax & Heuristic Guardrails:** Includes specialized JavaScript code nodes that run heuristic checks on the AI's output. If the payload contains phrases like "Here is the fix..." or lacks standard Python signatures (`def`, `class`, `import`, `print`), the workflow instantly throws an error to prevent bad commits.
* **Automated Retry Loop:** Features a built-in state machine and retry counter. If the Gemini model generates an invalid format on the first pass, the workflow can catch the error and execute a retry attempt before aborting.
* **Seamless Bi-Directional Sync:** After safely committing the encoded payload and opening a Pull Request, the agent automatically loops back to update the original Jira ticket with the status of the fix.

---

## Core Workflow Architecture
The pipeline is triggered by incoming webhooks from Jira and executes through four critical phases:

### Phase 1: Ingestion & Intent Parsing
* **Jira Trigger:** Listens specifically for `jira:issue_created` events.
* **Extract Issue Data:** Parses the Jira issue key, summary, and bug description. Dynamically generates standard Git branch names (e.g., `fix/proj-123`) and maps the configured GitHub organization, repository, and target file path (defaulted to `app/main.py`).

### Phase 2: Repository Analysis & Preparation
* **Fetch File from GitHub:** Makes an authenticated REST API call to GitHub to pull the exact state of the target file.
* **Decode File:** Base64 decodes the raw source code from the repository so it can be fed into the AI as clean context.
* **Build AI Prompt:** Constructs a highly restricted, zero-shot prompt. It combines the original code and the Jira bug description, appending non-negotiable rules for the AI (e.g., "NO explanations", "NO markdown", "Start on line 1 with Python code").

### Phase 3: AI Generation & Strict Validation
* **Call Google Gemini:** Sends the prompt to `gemini-flash-latest` using a low temperature (0.1) to guarantee highly deterministic, code-focused output.
* **Parse & Clean Response:** A dedicated Node.js step strips away any lingering `python` markdown formatting that the LLM might have stubbornly included.
* **Syntax Guardrails:** Evaluates the stripped string against Python structural regex checks and blocks the pipeline if conversational text is detected.
* **Re-Encode:** Base64 encodes the repaired, verified Python string for GitHub API compatibility.

### Phase 4: Version Control Deployment
* **Branch Creation:** Fetches the SHA of the `main` branch and creates a new Git reference (branch) for the specific Jira issue.
* **Commit Code:** Uses the GitHub Contents API to forcefully update the target file on the new branch with the AI-generated payload.
* **Create PR:** Automatically opens a Pull Request merging the `fix/*` branch into `main`, populating the PR body with the Jira ticket description.
* **Update Jira:** Syncs the automation state back to the original Jira issue.

---

## Tech Stack & Requirements

| Component | Technology / Platform |
| :--- | :--- |
| **Automation Engine** | n8n |
| **AI / LLM** | Google Gemini `gemini-flash-latest` |
| **Source Control** | GitHub REST API |
| **Issue Tracker** | Jira Software Cloud |
| **Scripting** | JavaScript (Node.js Code Nodes) |

---

## Getting Started

### Prerequisites
* **n8n Instance:** Self-hosted or Cloud version capable of importing JSON workflows.
* **GitHub Account:** A Personal Access Token (PAT) with repository scopes.
* **Google Gemini API Key:** Access to Google AI Studio for Gemini API credentials.
* **Jira Software Cloud:** Admin access to configure webhooks and an API token.

### Installation
1. Download the `workflow.json` file.
2. Open your n8n workspace, click **Add Workflow**, select the dropdown menu in the top right, and choose **Import from File**.
3. Select the downloaded JSON file.
4. Open the **Jira Issue Created** and **Update Jira** nodes to configure your `jiraSoftwareCloudApi` credentials.
5. Open the GitHub nodes (e.g., **Get File**, **Create Branch**, **Commit Code**) and select your `githubApi` credentials.
6. Open the **Call Google Gemini** node and insert your Gemini API Key in the query parameters.
7. Open the **Extract Issue Data** node and update `githubOrg`, `githubRepo`, and `filePath` to point to your target repository and file.
8. Activate the Workflow using the toggle in the top right of the n8n UI.

---

## Jira Configuration (Webhook Setup)
To trigger the automated pipeline, your n8n instance must be connected to your Jira Cloud environment.

1. **Create Webhook in Jira:** Navigate to **Jira Settings > System > Webhooks**. Click **Create a webhook**. Name it `n8n AI Automation`. Under **Events**, check the box for `Issue: created`.
2. **Set JQL Filter (Optional but Recommended):** To prevent the AI from trying to fix epic or non-code tasks, specify a JQL filter in the webhook setup (e.g., `issuetype = Bug AND project = "Backend"`).
3. **Connect to n8n:** Ensure your n8n **Jira Trigger** node is active and properly listening for the incoming payload.
