# Claude Code Action: A Beginner's Tutorial

This tutorial explains Claude Code Action for people with minimal GitHub and Claude experience. We'll start from the basics and build up to understanding how everything works together.

---

## Table of Contents

1. [What Problem Does This Solve?](#1-what-problem-does-this-solve)
2. [Understanding GitHub Basics](#2-understanding-github-basics)
3. [What is Claude?](#3-what-is-claude)
4. [What is a GitHub Action?](#4-what-is-a-github-action)
5. [What is Claude Code Action?](#5-what-is-claude-code-action)
6. [How It All Works Together](#6-how-it-all-works-together)
7. [Setting It Up Step by Step](#7-setting-it-up-step-by-step)
8. [Real Examples](#8-real-examples)
9. [Common Questions](#9-common-questions)
10. [Understanding @Mentions: GitHub vs GitHub Actions](#10-understanding-mentions-github-vs-github-actions)
11. [Deep Dive: Complete Workflow Trace](#11-deep-dive-complete-workflow-trace)

---

## 1. What Problem Does This Solve?

Imagine you're working on a software project with a team. Every day, you face tasks like:

- **Reviewing code** that teammates wrote (checking for bugs, style issues, security problems)
- **Answering questions** about how the code works
- **Fixing bugs** that people report
- **Writing documentation** for new features

These tasks take a lot of time. What if an AI assistant could help with them automatically?

**Claude Code Action** connects Claude (an AI assistant) to your GitHub project so Claude can:
- Automatically review code when someone submits changes
- Answer questions when someone asks in a comment
- Help fix issues that people report
- And much more

---

## 2. Understanding GitHub Basics

Before we dive into Claude Code Action, let's understand GitHub concepts.

### What is GitHub?

GitHub is a website where developers store and collaborate on code. Think of it like Google Docs, but for code:
- Multiple people can work on the same project
- Changes are tracked so you can see who changed what and when
- People can discuss changes before accepting them

### What is a Repository?

A **repository** (or "repo") is a project folder on GitHub. It contains:
- All the code files
- A history of every change ever made
- Discussions and issues

### What is an Issue?

An **issue** is like a support ticket or task. People create issues to:
- Report bugs ("The login button doesn't work")
- Request features ("Can we add dark mode?")
- Ask questions ("How do I configure the database?")

**Example of an issue:**
```
Title: Login button not working on mobile

Description: When I tap the login button on my iPhone, nothing happens.
I'm using Safari on iOS 17.

Steps to reproduce:
1. Go to the website on mobile
2. Tap "Login"
3. Nothing happens
```

### What is a Pull Request (PR)?

When someone wants to change the code, they don't change it directly. Instead, they:

1. Make a copy of the code (called a "branch")
2. Make their changes on that copy
3. Create a **Pull Request** asking to merge their changes into the main code

A Pull Request lets others review the changes before they're accepted.

**Example:**
- Alice wants to fix a typo in the README
- She creates a branch, fixes the typo, and opens a Pull Request
- Bob reviews it and says "Looks good!"
- The changes are merged into the main code

### What is a Comment?

Comments are messages people write on issues or pull requests. They're used for:
- Discussing the issue or changes
- Asking questions
- Giving feedback

### What is a Mention?

A **mention** is when you type `@username` in a comment to notify someone.

**Example:**
```
@alice Can you take a look at this bug? I think it's related to your recent changes.
```

When you mention someone:
- They get a notification (email, GitHub notification)
- Their username becomes a clickable link
- They know you want their attention

**This is key for Claude Code Action:** When you type `@claude` in a comment, it's like mentioning a team member named "Claude" - except Claude is an AI that will respond!

---

## 3. What is Claude?

### Claude: An AI Assistant

Claude is an AI assistant made by Anthropic. It can:
- Answer questions in natural language
- Write and explain code
- Review code for bugs and improvements
- Help with many text-based tasks

You might have used Claude at [claude.ai](https://claude.ai) where you chat with it in a browser.

### Claude Code

**Claude Code** is a version of Claude specifically designed to work with code. It can:
- Read and understand code files
- Edit code files
- Run commands (like tests)
- Understand the structure of a codebase

Think of Claude Code as a programmer AI that can actually make changes to your project, not just talk about code.

---

## 4. What is a GitHub Action?

### Automation on GitHub

A **GitHub Action** is a way to automatically run tasks when something happens in your repository.

**Real-world analogy:**
- Imagine you have a doorbell camera
- When someone rings the bell (trigger), it automatically records video (action)
- You set up the rule once, and it runs automatically every time

### How GitHub Actions Work

1. **Trigger:** Something happens (someone opens a PR, creates an issue, etc.)
2. **Workflow runs:** GitHub runs a script you defined
3. **Result:** Something happens (tests run, code is deployed, etc.)

### Where Are Actions Defined?

Actions are defined in YAML files inside a special folder:
```
your-repository/
├── .github/
│   └── workflows/
│       └── my-action.yml    <-- This file defines the action
├── src/
│   └── (your code files)
└── README.md
```

### Example: A Simple Action

Here's a simple action that runs tests whenever someone opens a Pull Request:

```yaml
# File: .github/workflows/run-tests.yml

name: Run Tests                    # Name of this action

on:                                # WHEN should this run?
  pull_request:                    # When a Pull Request is...
    types: [opened]                # ...opened

jobs:                              # WHAT should happen?
  test:                            # A job called "test"
    runs-on: ubuntu-latest         # Run on a Linux computer
    steps:                         # Steps to execute:
      - uses: actions/checkout@v4  # 1. Download the code
      - run: npm install           # 2. Install dependencies
      - run: npm test              # 3. Run tests
```

**What this does:**
1. Someone opens a Pull Request
2. GitHub sees the trigger matches (`pull_request` + `opened`)
3. GitHub starts a virtual computer (ubuntu-latest)
4. It downloads your code, installs dependencies, and runs tests
5. You see the results on the Pull Request page

---

## 5. What is Claude Code Action?

Now we can understand Claude Code Action!

### The Simple Explanation

**Claude Code Action** is a GitHub Action that:
1. Watches for triggers (mentions, new PRs, new issues)
2. Sends context to Claude (the code, the conversation, etc.)
3. Lets Claude respond or make changes
4. Posts Claude's response back to GitHub

### Visual Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         YOUR GITHUB REPOSITORY                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ Someone writes "@claude please
                                │ review this code"
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TRIGGER DETECTED                           │
│         GitHub sees a comment with "@claude" mention            │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    GITHUB ACTION STARTS                         │
│                                                                 │
│  1. Downloads your code                                         │
│  2. Gathers context (PR details, file changes, conversation)   │
│  3. Sends everything to Claude                                  │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      CLAUDE PROCESSES                           │
│                                                                 │
│  Claude reads the code, understands the request, and:          │
│  - Writes a response                                            │
│  - OR makes code changes                                        │
│  - OR does both                                                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RESULTS POSTED                               │
│                                                                 │
│  - Claude's response appears as a comment                       │
│  - If Claude made changes, a new branch is created             │
│  - You can review and accept the changes                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. How It All Works Together

### Scenario 1: Asking Claude a Question

**Situation:** You're confused about how a function works.

**What you do:**
1. Go to the Pull Request or Issue on GitHub
2. Write a comment: `@claude What does the calculateTax() function do?`
3. Submit the comment

**What happens behind the scenes:**
1. GitHub detects a new comment
2. The Claude Code Action workflow starts
3. It sees `@claude` in the comment (the trigger phrase)
4. It gathers context: the comment, the code files, the PR/issue details
5. It sends this to Claude with your question
6. Claude analyzes the code and writes an explanation
7. The action posts Claude's explanation as a reply comment

**What you see:**
```
You: @claude What does the calculateTax() function do?

Claude: The calculateTax() function calculates sales tax based on...
        [detailed explanation]
```

### Scenario 2: Asking Claude to Make Changes

**Situation:** You want Claude to fix a bug.

**What you do:**
1. On an issue, write: `@claude Can you fix this login bug? The button should redirect to /dashboard after login.`

**What happens:**
1. Claude reads the codebase and finds the login code
2. Claude edits the code to fix the bug
3. Claude creates a new branch with the fix
4. Claude opens a Pull Request with the changes
5. Claude comments with a link to the PR

**What you see:**
```
You: @claude Can you fix this login bug?

Claude: I've analyzed the issue and created a fix. The problem was in
        the handleLogin() function where the redirect was missing.

        I've created a Pull Request with the fix: #123
```

### Scenario 3: Automatic Code Review

**Situation:** You want every Pull Request to be reviewed automatically.

**Setup:** You configure the action to run whenever a PR is opened, with a prompt like "Review this code for bugs and security issues."

**What happens:**
1. Alice opens a Pull Request
2. The action automatically triggers (no `@claude` mention needed)
3. Claude reviews all the changed code
4. Claude posts comments about any issues found

**What you see:**
```
Claude: I've reviewed this Pull Request. Here are my findings:

        ⚠️ Security Issue (line 45): User input is not sanitized
        before being used in the SQL query. This could allow SQL
        injection attacks.

        💡 Suggestion (line 72): This loop could be simplified using
        the .map() function.

        ✅ Overall: The code structure is good, but please address
        the security issue before merging.
```

---

## 7. Setting It Up Step by Step

### Prerequisites

Before starting, you need:
1. A GitHub account
2. A repository where you have admin access
3. An Anthropic API key (get one at [console.anthropic.com](https://console.anthropic.com))

### Step 1: Install the Claude GitHub App

1. Go to: https://github.com/apps/claude
2. Click "Install"
3. Choose which repositories to install it on
4. Click "Install"

**Why this is needed:** The GitHub App gives Claude permission to read and write to your repository.

### Step 2: Add Your API Key as a Secret

Your API key is like a password - you never put it directly in code. Instead, use GitHub Secrets.

1. Go to your repository on GitHub
2. Click "Settings" (tab at the top)
3. Click "Secrets and variables" in the left sidebar
4. Click "Actions"
5. Click "New repository secret"
6. Name: `ANTHROPIC_API_KEY`
7. Value: Paste your API key from console.anthropic.com
8. Click "Add secret"

### Step 3: Create the Workflow File

1. In your repository, create a new file at:
   `.github/workflows/claude.yml`

2. Add this content:

```yaml
name: Claude Code Action

# WHEN should this run?
on:
  # When someone comments on an issue
  issue_comment:
    types: [created]

  # When someone comments on a PR
  pull_request_review_comment:
    types: [created]

  # When a new issue is created
  issues:
    types: [opened]

  # When a PR review is submitted
  pull_request_review:
    types: [submitted]

# WHAT should happen?
jobs:
  claude-response:
    # Only run if the comment contains @claude
    # (or if it's a new issue mentioning Claude)
    if: |
      contains(github.event.comment.body, '@claude') ||
      contains(github.event.issue.body, '@claude') ||
      contains(github.event.review.body, '@claude')

    runs-on: ubuntu-latest

    # Permissions the action needs
    permissions:
      contents: write       # To read/write code
      pull-requests: write  # To comment on PRs
      issues: write         # To comment on issues
      id-token: write       # For authentication

    steps:
      # Step 1: Download the code
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Get full history

      # Step 2: Run Claude Code Action
      - name: Run Claude
        uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

3. Commit this file to your repository

### Step 4: Test It!

1. Create a new Issue in your repository
2. In the issue body or a comment, write:
   ```
   @claude What files are in this repository?
   ```
3. Wait a minute - you'll see Claude respond!

---

## 8. Real Examples

### Example 1: Ask Claude About Code

**Your comment:**
```
@claude Can you explain how the authentication works in this project?
```

**Claude's response:**
```
I've analyzed the authentication system in this project. Here's how it works:

1. **Login Flow** (src/auth/login.ts)
   - User submits email and password
   - Server validates credentials against the database
   - If valid, a JWT token is generated and returned

2. **Token Storage** (src/auth/token.ts)
   - The JWT token is stored in localStorage
   - It's included in the Authorization header for API requests

3. **Protected Routes** (src/middleware/auth.ts)
   - The auth middleware checks for a valid token
   - If missing or expired, it returns a 401 error

Would you like me to explain any part in more detail?
```

### Example 2: Ask Claude to Fix a Bug

**Your comment:**
```
@claude The date is showing in the wrong format. It should be "March 15, 2024"
but it shows "2024-03-15". Can you fix this?
```

**Claude's response:**
```
I've found and fixed the date formatting issue. The problem was in
`src/utils/formatDate.ts` where the date was using ISO format instead of
a human-readable format.

I've created a pull request with the fix: #47

Changes made:
- Updated formatDate() to use toLocaleDateString() with appropriate options
- Added a test case for the new format

Let me know if you'd like any adjustments!
```

### Example 3: Code Review

**Your comment:**
```
@claude Can you review this PR for any issues?
```

**Claude's response:**
```
I've reviewed the changes in this Pull Request. Here's my analysis:

## Security Issues 🔴

**File: src/api/users.ts, Line 23**
```javascript
const query = `SELECT * FROM users WHERE id = ${userId}`;
```
This is vulnerable to SQL injection. Use parameterized queries instead:
```javascript
const query = `SELECT * FROM users WHERE id = $1`;
const result = await db.query(query, [userId]);
```

## Code Quality 🟡

**File: src/utils/helpers.ts, Line 45**
This function is quite long (87 lines). Consider breaking it into smaller functions.

## Positive Notes 🟢

- Good use of TypeScript types
- Clear variable naming
- Tests are included

## Summary

Please fix the SQL injection vulnerability before merging. The other items
are suggestions for improvement.
```

### Example 4: Automatic PR Review (No Mention Needed)

For fully automatic reviews, modify your workflow to trigger on all PRs:

```yaml
name: Auto Review PRs

on:
  pull_request:
    types: [opened, synchronize]  # Triggers when PR opened or updated

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write

    steps:
      - uses: actions/checkout@v4

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: |
            Please review this Pull Request. Focus on:
            - Security vulnerabilities
            - Potential bugs
            - Code style and readability

            Be constructive and helpful in your feedback.
```

Now every PR will be automatically reviewed by Claude!

---

## 9. Common Questions

### "How much does this cost?"

You need an Anthropic API key which has usage-based pricing. Each time Claude responds, you're charged based on the amount of text processed. Check [anthropic.com/pricing](https://www.anthropic.com/pricing) for current rates.

### "Can Claude access my private repository?"

Yes, but only if you install the Claude GitHub App on that repository. The App uses secure authentication (OIDC tokens) so your code is protected.

### "Can Claude make mistakes?"

Yes, Claude is AI and can make mistakes. Always review:
- Code changes Claude makes before merging
- Answers Claude gives for accuracy
- Security recommendations (get a human expert for critical systems)

### "How do I limit what Claude can do?"

Use the `claude_args` option to control Claude's capabilities:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    claude_args: |
      --allowedTools "Read,Grep,Glob"  # Only allow reading, no editing
```

### "Can I change the trigger from @claude to something else?"

Yes! Use the `trigger_phrase` option:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    trigger_phrase: "/ai"  # Now use /ai instead of @claude
```

### "What if Claude doesn't respond?"

Check:
1. Is the workflow file in `.github/workflows/`?
2. Did you add the `ANTHROPIC_API_KEY` secret?
3. Is the Claude GitHub App installed on the repository?
4. Did you include the trigger phrase (`@claude` by default)?
5. Check the Actions tab for error messages

### "Can I use this with AWS or Google Cloud?"

Yes! Instead of the Anthropic API, you can use:
- **AWS Bedrock:** Set `use_bedrock: "true"`
- **Google Vertex AI:** Set `use_vertex: "true"`

This requires additional AWS/GCP authentication setup.

---

## 10. Understanding @Mentions: GitHub vs GitHub Actions

A common question is: "How does GitHub Actions know how to contact alice (and which alice) when `@alice` is used?"

The answer reveals an important distinction between two completely different systems.

### GitHub's Built-in @Mention System

When you type `@alice` in a comment, **GitHub itself** (not GitHub Actions) handles the notification:

1. GitHub parses the comment text looking for `@username` patterns
2. It looks up `alice` in its user database for that repository/organization
3. It sends a notification (email, GitHub notification) to that user
4. The username becomes a clickable link to their profile

This is native GitHub functionality - no Actions involved.

### How GitHub Actions Detects @claude

GitHub Actions doesn't "contact" anyone. It simply:

1. **Receives the event payload** - When a comment is created, GitHub sends the full comment text to any triggered workflow
2. **Pattern matches on the text** - The workflow checks if the string `@claude` appears:

```yaml
if: contains(github.event.comment.body, '@claude')
```

3. **Runs the action** - If matched, the Claude Code Action runs and posts a response

### Key Distinction

| Aspect | @alice (human) | @claude (action) |
|--------|---------------|------------------|
| Who handles it | GitHub platform | GitHub Actions workflow |
| What happens | Notification sent to user | Workflow triggered, AI responds |
| User must exist? | Yes, in GitHub's database | No - it's just a text pattern |

The `@claude` in Claude Code Action is essentially a **convention** - you could change it to `/ai` or `hey-claude` with the `trigger_phrase` option. It's just string matching, not GitHub's mention system.

### Why This Matters

Understanding this distinction helps you realize:

- **`@claude` is not a real GitHub user** - It's a trigger phrase that your workflow looks for
- **You can customize it** - Change `@claude` to any text pattern you prefer
- **Multiple triggers are possible** - You could set up different actions for `@claude-review`, `@claude-fix`, etc.
- **It's just text matching** - The workflow file defines what text triggers the action

---

## 11. Deep Dive: Complete Workflow Trace

Let's trace exactly what happens when you type `@claude, please resolve issue #2` in a GitHub comment. This deep dive shows every system involved and how they communicate.

### Step 1: You Submit the Comment

```
You type: "@claude, please resolve issue #2"
You click: Submit
```

**What happens:** Your browser sends an HTTP POST request to GitHub's servers with the comment text.

### Step 2: GitHub Platform Processes the Comment

**GitHub's servers:**

1. **Store the comment** in their database
2. **Parse @mentions** - sees `@claude`, but there's no GitHub user named "claude" in this context (it's just text)
3. **Emit a webhook event** - GitHub fires an `issue_comment` event to all configured listeners

**The webhook payload looks like:**
```json
{
  "action": "created",
  "comment": {
    "body": "@claude, please resolve issue #2",
    "user": { "login": "your-username" },
    "created_at": "2025-12-29T..."
  },
  "issue": { "number": 5 },
  "repository": { "full_name": "owner/repo" }
}
```

### Step 3: GitHub Actions Receives the Event

GitHub Actions is a **separate system** within GitHub that listens for repository events.

1. **Event matching** - GitHub Actions checks: "Are there any workflow files in `.github/workflows/` that listen to `issue_comment` events?"

2. **Finds your workflow file** (`.github/workflows/claude.yml`):
```yaml
on:
  issue_comment:
    types: [created]
```

3. **Evaluates the `if` condition**:
```yaml
if: contains(github.event.comment.body, '@claude')
```
   - `github.event.comment.body` = `"@claude, please resolve issue #2"`
   - `contains(..., '@claude')` = **true**

4. **Spins up a runner** - GitHub provisions a fresh Ubuntu virtual machine

### Step 4: GitHub Actions Executes the Workflow

On the runner VM, GitHub Actions executes each step:

**Step 4a: Checkout**
```yaml
- uses: actions/checkout@v4
```
- Downloads your repository code to the runner

**Step 4b: Run Claude Code Action**
```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**Where does `anthropics/claude-code-action@v1` come from?**

- It's a **public GitHub repository**: `github.com/anthropics/claude-code-action`
- The `@v1` is a git tag/release
- GitHub Actions **downloads** this action's code to the runner
- It then **executes** it (it's a JavaScript/TypeScript action using Bun)

### Step 5: Claude Code Action - Phase 1 (Preparation)

The action's preparation phase handles setup:

```
┌─────────────────────────────────────────────────────────────┐
│                    PREPARATION PHASE                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. AUTHENTICATION SETUP                                    │
│     - Uses OIDC token exchange with GitHub                  │
│     - Authenticates with Claude GitHub App                  │
│     - Validates ANTHROPIC_API_KEY                           │
│                                                             │
│  2. PERMISSION VALIDATION                                   │
│     - Checks: Does the commenter have write access?         │
│     - Checks: Is this a human (not a bot)?                  │
│                                                             │
│  3. TRIGGER DETECTION                                       │
│     - Tag Mode: Confirms "@claude" is in the comment        │
│     - Extracts the prompt: "please resolve issue #2"        │
│                                                             │
│  4. CONTEXT CREATION                                        │
│     - Fetches issue #2 data via GitHub API                  │
│     - Fetches current issue/PR context                      │
│     - Creates a "tracking comment" (the progress indicator) │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Step 6: Claude Code Action - Phase 2 (Execution)

The execution phase is where Claude actually does the work:

```
┌─────────────────────────────────────────────────────────────┐
│                    EXECUTION PHASE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. MCP SERVER SETUP                                        │
│     - Installs GitHub MCP servers for Claude to use:        │
│       • github-actions-server (workflow access)             │
│       • github-comment-server (comment operations)          │
│       • github-file-ops-server (file access)                │
│                                                             │
│  2. PROMPT GENERATION                                       │
│     Builds a prompt like:                                   │
│     ┌─────────────────────────────────────────────────┐     │
│     │ User request: "please resolve issue #2"         │     │
│     │                                                 │     │
│     │ Context:                                        │     │
│     │ - Current issue: #5                             │     │
│     │ - Referenced issue #2:                          │     │
│     │   Title: "Login button broken"                  │     │
│     │   Body: "When I click login..."                 │     │
│     │                                                 │     │
│     │ Repository files: [list of files]               │     │
│     └─────────────────────────────────────────────────┘     │
│                                                             │
│  3. CLAUDE API CALL  ◄── This is where Claude is invoked   │
│     - HTTP POST to https://api.anthropic.com/v1/messages    │
│     - Sends: prompt, context, available tools               │
│     - Auth: Your ANTHROPIC_API_KEY                          │
│                                                             │
│  4. CLAUDE PROCESSES (on Anthropic's servers)               │
│     - Claude reads the context                              │
│     - Claude can use tools:                                 │
│       • Read files from your repo                           │
│       • Edit files                                          │
│       • Run bash commands                                   │
│       • Search code                                         │
│     - Claude generates a solution                           │
│                                                             │
│  5. RESULT PROCESSING                                       │
│     - If Claude edited files → create a new branch + PR     │
│     - Update the tracking comment with results              │
│     - Post Claude's response as a comment                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Step 7: Results Posted to GitHub

The action uses the GitHub API to:

1. **Update the tracking comment** with completion status
2. **Post Claude's response** as a new comment
3. **Create a PR** (if Claude made code changes)

### Visual Summary: The Complete Flow

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────────┐
│    YOU       │     │     GITHUB       │     │   GITHUB ACTIONS    │
│              │     │    (Platform)    │     │     (Runner VM)     │
└──────┬───────┘     └────────┬─────────┘     └──────────┬──────────┘
       │                      │                          │
       │ POST comment         │                          │
       │─────────────────────►│                          │
       │                      │                          │
       │                      │ Webhook: issue_comment   │
       │                      │─────────────────────────►│
       │                      │                          │
       │                      │                          │ Download action from
       │                      │                          │ anthropics/claude-code-action
       │                      │                          │
       │                      │         ┌────────────────┴────────────────┐
       │                      │         │     CLAUDE CODE ACTION          │
       │                      │         │         (Running)               │
       │                      │         └────────────────┬────────────────┘
       │                      │                          │
       │                      │                          │ API Call
       │                      │                          ▼
       │                      │         ┌─────────────────────────────────┐
       │                      │         │      ANTHROPIC API              │
       │                      │         │   (api.anthropic.com)           │
       │                      │         │                                 │
       │                      │         │   Claude processes request,     │
       │                      │         │   reads code, generates fix     │
       │                      │         └────────────────┬────────────────┘
       │                      │                          │
       │                      │                          │ Response
       │                      │                          ▼
       │                      │         ┌─────────────────────────────────┐
       │                      │         │  Action creates branch + PR     │
       │                      │◄────────│  Action posts comment           │
       │                      │         └─────────────────────────────────┘
       │                      │
       │  See comment + PR    │
       │◄─────────────────────│
       │                      │
```

### Key Clarifications

| Question | Answer |
|----------|--------|
| **Where is Claude Code?** | It's Anthropic's AI, accessed via API at `api.anthropic.com` |
| **Where is Claude Code Action?** | A GitHub repo: `github.com/anthropics/claude-code-action` |
| **Who runs the action?** | GitHub Actions (on their runner VMs) |
| **Who runs Claude?** | Anthropic (on their servers) |
| **How does the action invoke Claude?** | HTTP API call using your `ANTHROPIC_API_KEY` |

### The Four Players

Understanding the complete system requires recognizing four distinct players:

1. **GitHub (Platform)** - Stores your code, handles comments, sends webhook events
2. **GitHub Actions (Automation)** - Listens for events, runs workflows on VMs
3. **Claude Code Action (Bridge)** - Downloaded and run by GitHub Actions, translates between GitHub and Claude
4. **Anthropic API (AI)** - Receives requests, runs Claude, returns intelligent responses

The action is essentially a **bridge** that translates GitHub events into Claude API calls and Claude's responses back into GitHub actions (comments, PRs, etc.).

---

## Summary

**Claude Code Action** connects Claude AI to your GitHub repository so that:

1. **You can talk to Claude** by writing `@claude` in comments
2. **Claude can help with tasks** like explaining code, reviewing PRs, and fixing bugs
3. **Automation is possible** by configuring workflows to run on specific triggers

**The key pieces are:**
- **GitHub Actions:** Automation that runs when things happen in your repo
- **Claude Code:** An AI that understands and can modify code
- **The Claude GitHub App:** Gives Claude permission to access your repository
- **Your workflow file:** Tells GitHub when to run Claude and what to do

Start simple with the basic workflow, then explore more advanced features as you get comfortable!

---

## Next Steps

1. Set up the basic workflow following Step 7
2. Try asking Claude questions about your code
3. Explore the `examples/` folder for more workflow templates
4. Read `docs/usage.md` for all configuration options

Happy coding with Claude!
