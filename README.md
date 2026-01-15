# GitHub Automatic PR Review Workflow (AI-Powered)

## 1. Overview

<img width="1048" height="420" alt="github pr" src="https://github.com/user-attachments/assets/b7229219-7d08-401d-8e2e-5a33f39eff6d" />


The **GitHub Automatic PR Review Workflow** is an **n8n-based automation** that performs **AI-driven code reviews** whenever a pull request (PR) is opened or updated in a GitHub repository.

The workflow:

* Listens for GitHub pull request events
* Fetches file diffs from the PR
* Converts the diffs into a structured prompt
* Uses an AI agent to review the code
* Posts the AI-generated review as a PR comment
* Optionally labels the PR as reviewed by AI

This automation improves **developer productivity**, **review consistency**, and **code quality** by providing instant feedback on pull requests.

---

## 2. High-Level Architecture

**Trigger → Data Extraction → Prompt Engineering → AI Review → GitHub Feedback**

```
GitHub PR Event
      ↓
Fetch PR File Diffs
      ↓
Create Structured AI Prompt
      ↓
AI Code Review Agent
      ↓
Post PR Comment
      ↓
Add "ReviewedByAI" Label
```

---

## 3. Workflow Trigger

### 3.1 GitHub Trigger Node – `PR Trigger`

**Node Type:** `n8n-nodes-base.githubTrigger`
**Event:** `pull_request`

#### Purpose

This node initiates the workflow whenever a **pull request event** occurs in the configured GitHub repository.

#### Key Configuration

* **Authentication:** GitHub OAuth2
* **Repository Owner:** `mayank953`
* **Repository Name:** `n8n-demo-pr`
* **Subscribed Events:** `pull_request`

#### Output

The trigger emits the full GitHub webhook payload, including:

* PR number
* Repository details
* Sender information
* PR metadata (title, body, changed files count)

---

## 4. Fetching Pull Request Diffs

### 4.1 HTTP Request Node – `Get file's Diffs from PR`

**Node Type:** `n8n-nodes-base.httpRequest`

#### Purpose

Retrieves the list of files modified in the pull request along with their **diff patches**.

#### GitHub API Endpoint

```
https://api.github.com/repos/{owner}/{repo}/pulls/{pull_number}/files
```

Dynamic values are injected from the PR trigger payload:

* `sender.login`
* `repository.name`
* `pull_request.number`

#### Output

An array of file objects containing:

* `filename`
* `status` (modified, added, removed)
* `patch` (unified diff, if available)

Binary files may not include a `patch`.

---

## 5. Prompt Engineering

### 5.1 Code Node – `Create target Prompt from PR Diffs`

**Node Type:** `n8n-nodes-base.code`

#### Purpose

Transforms raw GitHub diff data into a **clean, structured AI prompt** suitable for automated code review.

#### Key Processing Logic

* Iterates through all changed files
* Skips files without patches (e.g., binaries)
* Formats diffs using Markdown `diff` blocks
* Escapes triple backticks to avoid prompt corruption
* Builds a single consolidated **user message**

#### Prompt Structure

The generated prompt instructs the AI to:

* Act as a **senior developer**
* Review code file-by-file
* Focus on meaningful changes
* Provide **inline review comments**
* Avoid repeating filenames or code snippets

#### Output

```json
{
  "user_message": "You are a senior Python developer. Please review the following code changes..."
}
```

This message is passed directly to the AI agent.

---

## 6. AI Code Review

### 6.1 OpenAI Chat Model Node – `OpenAI Chat Model`

**Node Type:** `lmChatOpenAi`
**Model:** `gpt-4o`

#### Purpose

Provides the underlying language model used by the AI agent to reason about the code changes.

#### Role

* Acts as the **LLM backbone**
* Receives structured instructions and diffs
* Generates natural-language review feedback

---

### 6.2 LangChain Agent Node – `Code Review Agent`

**Node Type:** `@n8n/n8n-nodes-langchain.agent`

#### Purpose

Executes the AI-based reasoning and produces the final **code review comments**.

#### Input

* `user_message` (generated from PR diffs)
* Connected LLM (OpenAI Chat Model)

#### Output

A natural language response containing:

* Inline-style review comments
* Best practice suggestions
* Potential issues or improvements

---

## 7. Posting Review to GitHub

### 7.1 GitHub Node – `GitHub Robot`

**Node Type:** `n8n-nodes-base.github`
**Operation:** Create Review Comment

#### Purpose

Posts the AI-generated review as a **comment on the pull request**.

#### Key Configuration

* **Resource:** `review`
* **Event Type:** `comment`
* **PR Number:** Retrieved dynamically from trigger
* **Comment Body:** AI agent output

#### Result

The PR receives an automated review comment visible to all collaborators.

---

## 8. PR Labeling (Optional)

### 8.1 GitHub Node – `Add Label to PR`

**Node Type:** `n8n-nodes-base.github`
**Operation:** Edit Issue

#### Purpose

Adds a label to indicate the PR has been reviewed by AI.

#### Label Applied

```
ReviewedByAI
```

#### Benefits

* Visual indicator of automated review completion
* Helps teams filter and track AI-reviewed PRs
* Can be used in CI/CD or governance workflows

---

## 9. Optional Extensions

### 9.1 Coding Guidelines Integration

The workflow supports extending the AI agent with:

* Google Sheets
* Databases
* Internal coding standards

This enables **opinionated reviews** aligned with team or company best practices.

---

## 10. Security & Best Practices

* OAuth2 authentication ensures secure GitHub access
* No code is modified automatically—only comments are posted
* Diff-based review minimizes data exposure
* AI output is transparent and auditable in PR history

---

## 11. Use Cases

* Automated first-pass code reviews
* Reducing reviewer workload
* Enforcing coding standards
* Improving PR turnaround time
* AI-assisted DevOps pipelines

---

## 12. Summary

This **GitHub Automatic PR Workflow** demonstrates a **production-grade AI integration** using:

* GitHub Webhooks
* REST APIs
* Prompt engineering
* LangChain agents
* OpenAI models
* n8n orchestration

It is a strong example of **AI-powered developer tooling** and can be extended for CI/CD, security reviews, or compliance checks.

---

If you want, I can:

* Convert this into **Markdown for GitHub**
* Simplify it for **non-technical stakeholders**
* Add **architecture diagrams**
* Rewrite it as a **portfolio case study**
