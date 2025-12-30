# Claude Code Action: Codebase Guide for Traditional Developers

> **Audience:** Developers familiar with C, C++, or Java who are new to full-stack web development, modern JavaScript/TypeScript, and AI-assisted programming.

> **Source:** This codebase is from [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action).

---

## Table of Contents

1. [Foundational Concepts](#1-foundational-concepts)
2. [Technology Stack Explained](#2-technology-stack-explained)
3. [Project Structure](#3-project-structure)
4. [Architecture Overview](#4-architecture-overview)
5. [Key Components Deep Dive](#5-key-components-deep-dive)
6. [Data Flow](#6-data-flow)
7. [Configuration Files Explained](#7-configuration-files-explained)
8. [Comparing to Traditional Development](#8-comparing-to-traditional-development)
9. [Building and Running](#9-building-and-running)
10. [Glossary](#10-glossary)

---

## 1. Foundational Concepts

Before diving into the code, let's establish some foundational concepts that differ from traditional C/C++/Java development.

### 1.1 Server-Side JavaScript (Node.js)

In traditional development, you might use:
- **C/C++:** Compiled to machine code, runs directly on the OS
- **Java:** Compiled to bytecode, runs on the JVM

**Node.js** is a runtime that lets you run JavaScript outside the browser:
- JavaScript code is interpreted/JIT-compiled by the V8 engine
- Single-threaded with an event loop (non-blocking I/O)
- Uses `npm` (Node Package Manager) for dependency management

```
Traditional:                    Node.js:
┌─────────────┐                ┌─────────────┐
│  C++ Code   │                │   JS Code   │
└──────┬──────┘                └──────┬──────┘
       │ compile                      │ interpret
       ▼                              ▼
┌─────────────┐                ┌─────────────┐
│Machine Code │                │  V8 Engine  │
└──────┬──────┘                └──────┬──────┘
       │                              │
       ▼                              ▼
┌─────────────┐                ┌─────────────┐
│     OS      │                │   Node.js   │
└─────────────┘                └─────────────┘
```

### 1.2 TypeScript

TypeScript is JavaScript with static types. Think of it as:
- **JavaScript** ≈ Python (dynamic typing)
- **TypeScript** ≈ Java (static typing, but compiles to JavaScript)

```typescript
// TypeScript (this codebase)
function add(a: number, b: number): number {
  return a + b;
}

// Equivalent Java
public int add(int a, int b) {
  return a + b;
}
```

TypeScript files (`.ts`) are **transpiled** to JavaScript (`.js`) before execution.

### 1.3 Bun Runtime

This project uses **Bun** instead of Node.js. Bun is:
- A faster alternative to Node.js (written in Zig)
- Includes a package manager, bundler, and test runner
- Can run TypeScript directly (no transpilation step needed)

```bash
# Node.js way
npm install           # Install dependencies
npx tsc              # Compile TypeScript
node dist/index.js   # Run compiled JavaScript

# Bun way
bun install          # Install dependencies
bun run src/index.ts # Run TypeScript directly
```

### 1.4 Asynchronous Programming

Unlike C/Java's thread-based concurrency, JavaScript uses **async/await**:

```typescript
// TypeScript async pattern
async function fetchData(): Promise<string> {
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();
  return data.message;
}

// Equivalent Java (conceptually)
public CompletableFuture<String> fetchData() {
  return httpClient.sendAsync(request, BodyHandlers.ofString())
    .thenApply(response -> parseJson(response.body()).getMessage());
}
```

Key differences:
- **No threads:** Single-threaded, uses event loop
- **Non-blocking:** I/O operations don't block the thread
- **Promise-based:** Async operations return Promises (like Java's Future)

### 1.5 What is an API?

An **API (Application Programming Interface)** in web development typically means:

- **REST API:** HTTP endpoints that accept requests and return JSON
- **GraphQL API:** Query language for APIs (used in this codebase for GitHub)

```
Your Code                    API Server
    │                            │
    │  HTTP POST /api/users      │
    │  {"name": "Alice"}         │
    │ ──────────────────────────►│
    │                            │ Process request
    │  HTTP 200 OK               │
    │  {"id": 1, "name": "Alice"}│
    │ ◄──────────────────────────│
```

### 1.6 What is AI-Assisted Programming?

This codebase integrates with **Claude**, an AI assistant:

1. **Your code** sends a prompt (text instruction) to Claude's API
2. **Claude's API** processes the prompt using a large language model (LLM)
3. **Claude responds** with text, code, or structured data

```
Your Application              Claude API (Anthropic)
    │                              │
    │  POST /v1/messages           │
    │  "Please review this code"   │
    │ ────────────────────────────►│
    │                              │ LLM processes
    │  Response:                   │
    │  "I found 3 issues..."       │
    │ ◄────────────────────────────│
```

---

## 2. Technology Stack Explained

### 2.1 Core Technologies

| Technology | Purpose | Traditional Equivalent |
|------------|---------|----------------------|
| **TypeScript** | Programming language | Java/C++ |
| **Bun** | Runtime & package manager | JVM + Maven/Gradle |
| **GitHub Actions** | CI/CD automation | Jenkins, TeamCity |
| **REST/GraphQL APIs** | Network communication | HTTP libraries, RPC |
| **JSON** | Data serialization | XML, Protocol Buffers |
| **YAML** | Configuration files | Properties files, XML config |

### 2.2 Key Dependencies

From `package.json`:

```json
{
  "dependencies": {
    "@actions/core": "^1.10.1",        // GitHub Actions SDK
    "@actions/github": "^6.0.1",       // GitHub API wrapper
    "@anthropic-ai/claude-agent-sdk": "^0.1.76",  // Claude AI SDK
    "@modelcontextprotocol/sdk": "^1.11.0",       // MCP for AI tools
    "@octokit/rest": "^21.1.1",        // GitHub REST API client
    "@octokit/graphql": "^8.2.2",      // GitHub GraphQL client
    "zod": "^3.24.4"                   // Runtime type validation
  }
}
```

**Analogy to Java:**
```
@actions/core       ≈  Maven plugin APIs
@octokit/rest       ≈  HttpClient + JSON parser
zod                 ≈  Bean Validation (JSR 380)
```

### 2.3 Understanding package.json

`package.json` is like a `pom.xml` (Maven) or `build.gradle`:

```json
{
  "name": "@anthropic-ai/claude-code-action",  // Project identifier
  "version": "1.0.0",                          // Version number
  "scripts": {                                 // Like Makefile targets
    "test": "bun test",                        // Run tests
    "format": "prettier --write .",            // Format code
    "typecheck": "tsc --noEmit"                // Type check
  },
  "dependencies": { ... },                     // Runtime dependencies
  "devDependencies": { ... }                   // Build-time dependencies
}
```

---

## 3. Project Structure

```
claude-code-action/
├── action.yml                 # GitHub Action definition (entry point)
├── package.json               # Dependencies and scripts
├── tsconfig.json              # TypeScript compiler configuration
├── src/                       # Main source code
│   ├── entrypoints/           # Entry points for different execution phases
│   │   ├── prepare.ts         # Phase 1: Setup and validation
│   │   ├── format-turns.ts    # Format Claude's conversation output
│   │   └── update-comment-link.ts  # Update GitHub comments
│   ├── github/                # GitHub integration layer
│   │   ├── api/               # API clients (REST, GraphQL)
│   │   ├── data/              # Data fetching and formatting
│   │   ├── operations/        # GitHub operations (branches, comments)
│   │   ├── validation/        # Permission and trigger validation
│   │   ├── context.ts         # Parse GitHub event context
│   │   ├── token.ts           # Token management
│   │   └── types.ts           # Type definitions
│   ├── modes/                 # Execution modes
│   │   ├── tag/               # @claude mention mode
│   │   ├── agent/             # Automation mode
│   │   ├── registry.ts        # Mode selection logic
│   │   └── types.ts           # Mode interfaces
│   ├── mcp/                   # Model Context Protocol servers
│   │   ├── github-actions-server.ts   # CI/CD access
│   │   ├── github-comment-server.ts   # Comment operations
│   │   └── github-file-ops-server.ts  # File operations
│   ├── create-prompt/         # Prompt generation
│   └── prepare/               # Preparation orchestration
├── base-action/               # Core Claude execution logic
│   ├── src/
│   │   ├── index.ts           # Main entry point
│   │   ├── run-claude.ts      # Execute Claude CLI
│   │   ├── prepare-prompt.ts  # Prepare prompts
│   │   └── validate-env.ts    # Environment validation
│   └── test/                  # Unit tests
├── test/                      # Integration tests
├── docs/                      # Documentation
└── examples/                  # Example workflow files
```

### 3.1 Comparing to a Traditional Project Structure

```
Traditional Java Project          This TypeScript Project
─────────────────────────         ─────────────────────────
pom.xml                           package.json
src/main/java/                    src/
src/test/java/                    test/
target/                           (no build output - Bun runs .ts directly)
application.properties            action.yml, .env files
```

---

## 4. Architecture Overview

### 4.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              GITHUB                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────────┐ │
│  │   Issues    │    │Pull Requests│    │    GitHub Actions Runner    │ │
│  │             │    │             │    │  ┌─────────────────────────┐│ │
│  │  @claude    │───►│  Webhook    │───►│  │  Claude Code Action     ││ │
│  │  comments   │    │  Events     │    │  │                         ││ │
│  └─────────────┘    └─────────────┘    │  │  Phase 1: Prepare       ││ │
│                                        │  │  Phase 2: Execute       ││ │
│                                        │  │  Phase 3: Update        ││ │
│                                        │  └───────────┬─────────────┘│ │
│                                        └──────────────┼──────────────┘ │
└───────────────────────────────────────────────────────┼────────────────┘
                                                        │
                                                        │ API Calls
                                                        ▼
                                        ┌───────────────────────────────┐
                                        │        ANTHROPIC API          │
                                        │                               │
                                        │   Claude AI processes the     │
                                        │   request and returns a       │
                                        │   response                    │
                                        └───────────────────────────────┘
```

### 4.2 Two-Phase Execution Model

The action runs in two main phases:

```
┌─────────────────────────────────────────────────────────────────┐
│                        PHASE 1: PREPARE                         │
│                   (src/entrypoints/prepare.ts)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Parse GitHub Context                                        │
│     └─► Determine event type (issue, PR, comment, etc.)        │
│                                                                 │
│  2. Detect Execution Mode                                       │
│     └─► "tag" mode (for @claude mentions)                      │
│     └─► "agent" mode (for automation)                          │
│                                                                 │
│  3. Setup Authentication                                        │
│     └─► Exchange OIDC token for GitHub App token               │
│     └─► Validate permissions                                    │
│                                                                 │
│  4. Check Trigger Conditions                                    │
│     └─► Does comment contain "@claude"?                        │
│     └─► Is user authorized?                                    │
│                                                                 │
│  5. Create Tracking Comment                                     │
│     └─► Post initial "Claude is working..." comment            │
│                                                                 │
│  6. Prepare Branch                                              │
│     └─► Create branch for Claude's changes                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       PHASE 2: EXECUTE                          │
│                   (base-action/src/index.ts)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Validate Environment                                        │
│     └─► Check required environment variables                   │
│                                                                 │
│  2. Setup Claude Code Settings                                  │
│     └─► Configure Claude's behavior                            │
│                                                                 │
│  3. Install MCP Servers                                         │
│     └─► Enable Claude to interact with GitHub                  │
│                                                                 │
│  4. Prepare Prompt                                              │
│     └─► Build context-rich prompt from GitHub data             │
│                                                                 │
│  5. Run Claude                                                  │
│     └─► Execute Claude CLI with prompt                         │
│     └─► Stream output to log file                              │
│                                                                 │
│  6. Process Results                                             │
│     └─► Parse Claude's response                                │
│     └─► Create PR if code was changed                          │
│     └─► Update tracking comment                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.3 Mode System

The action supports multiple "modes" of operation:

```
┌─────────────────────────────────────────────────────────────────┐
│                         MODE REGISTRY                           │
│                      (src/modes/registry.ts)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────┐    ┌──────────────────────┐          │
│  │      TAG MODE        │    │     AGENT MODE       │          │
│  │  (src/modes/tag/)    │    │  (src/modes/agent/)  │          │
│  ├──────────────────────┤    ├──────────────────────┤          │
│  │                      │    │                      │          │
│  │ Triggers on:         │    │ Triggers on:         │          │
│  │ • @claude mentions   │    │ • Explicit prompts   │          │
│  │ • Issue assignments  │    │ • Automation events  │          │
│  │                      │    │ • Scheduled runs     │          │
│  │ Features:            │    │                      │          │
│  │ • Progress tracking  │    │ Features:            │          │
│  │ • Interactive chat   │    │ • Silent execution   │          │
│  │ • Comment updates    │    │ • Structured output  │          │
│  │                      │    │ • CI/CD integration  │          │
│  └──────────────────────┘    └──────────────────────┘          │
│                                                                 │
│  Mode Selection Logic:                                          │
│  ─────────────────────                                          │
│  if (explicit_prompt_provided) → AGENT MODE                     │
│  else if (entity_event)        → TAG MODE                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Key Components Deep Dive

### 5.1 GitHub Context Parser (`src/github/context.ts`)

This module parses the GitHub event payload to understand what triggered the action.

**Concept:** When something happens on GitHub (comment, PR, issue), GitHub sends a JSON payload describing the event. This module parses that payload.

```typescript
// Simplified example of context parsing
export function parseGitHubContext(): GitHubContext {
  const eventName = process.env.GITHUB_EVENT_NAME;  // e.g., "issue_comment"
  const payload = github.context.payload;           // JSON from GitHub

  return {
    eventName,
    repository: {
      owner: payload.repository.owner.login,
      repo: payload.repository.name,
    },
    actor: payload.sender.login,
    // ... more fields
  };
}
```

**Traditional equivalent (Java):**
```java
public class GitHubContextParser {
    public GitHubContext parse() {
        String eventName = System.getenv("GITHUB_EVENT_NAME");
        JsonObject payload = readPayloadFile();

        return new GitHubContext(
            eventName,
            payload.getString("repository.owner.login"),
            payload.getString("repository.name"),
            payload.getString("sender.login")
        );
    }
}
```

### 5.2 Token Management (`src/github/token.ts`)

Handles authentication with GitHub using OIDC (OpenID Connect) tokens.

**Concept:** Instead of storing long-lived credentials, the action:
1. Gets a short-lived OIDC token from GitHub Actions
2. Exchanges it for a GitHub App installation token
3. Uses that token for API calls

```
GitHub Actions Runner          GitHub's Token Endpoint
        │                              │
        │  "I am workflow X in repo Y" │
        │  (OIDC Token)                │
        │ ────────────────────────────►│
        │                              │
        │  "Here's your access token"  │
        │  (GitHub App Token)          │
        │ ◄────────────────────────────│
        │                              │
        ▼
   Use token for GitHub API calls
```

### 5.3 MCP Servers (`src/mcp/`)

**MCP (Model Context Protocol)** is a standard for giving AI models access to external tools.

Think of MCP servers as **plugins** that extend Claude's capabilities:

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLAUDE AI                               │
│  "I need to update a GitHub comment"                            │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            │ MCP Protocol
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    MCP SERVER                                   │
│             (github-comment-server.ts)                          │
├─────────────────────────────────────────────────────────────────┤
│  Available Tools:                                               │
│  • update_claude_comment(body: string)                          │
│                                                                 │
│  Implementation:                                                │
│  1. Receives tool call from Claude                              │
│  2. Calls GitHub API to update comment                          │
│  3. Returns result to Claude                                    │
└─────────────────────────────────────────────────────────────────┘
```

**Analogy:** MCP is like Java's SPI (Service Provider Interface) - a way to dynamically extend functionality.

### 5.4 Prompt Generation (`src/create-prompt/`)

Builds the prompt that's sent to Claude, including:
- The user's request
- Relevant context (PR diff, issue description, code files)
- System instructions

```typescript
// Simplified prompt structure
const prompt = `
## User Request
${userComment}

## Context
Repository: ${repo.owner}/${repo.name}
PR #${prNumber}: ${prTitle}

## Changed Files
${prDiff}

## Instructions
Please review this code and provide feedback.
`;
```

---

## 6. Data Flow

### 6.1 Complete Request Flow

```
User writes "@claude please review this PR"
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────┐
│ 1. GITHUB EVENT                                                 │
│    GitHub creates issue_comment event                           │
│    Sends webhook to GitHub Actions                              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. WORKFLOW TRIGGERED                                           │
│    .github/workflows/claude.yml matches event                   │
│    Runner VM starts                                             │
│    action.yml steps begin                                       │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. PREPARE PHASE (prepare.ts)                                   │
│    • Parse context                                              │
│    • Validate permissions                                       │
│    • Check trigger phrase                                       │
│    • Create tracking comment                                    │
│    • Setup branch                                               │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. EXECUTE PHASE (base-action/src/index.ts)                     │
│    • Validate environment                                       │
│    • Setup MCP servers                                          │
│    • Prepare prompt with context                                │
│    • Call Claude CLI                                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. CLAUDE PROCESSES                                             │
│    • Reads the prompt                                           │
│    • Analyzes the code                                          │
│    • Uses MCP tools (read files, search, etc.)                  │
│    • Generates response                                         │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. RESULT PROCESSING                                            │
│    • Parse Claude's output                                      │
│    • If code changed: create branch, open PR                    │
│    • Update tracking comment with results                       │
│    • Post response as GitHub comment                            │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
        User sees Claude's response on GitHub
```

### 6.2 Data Structures

**GitHub Context (from webhook):**
```typescript
interface GitHubContext {
  eventName: "issue_comment" | "pull_request" | ...;
  repository: {
    owner: string;   // "anthropics"
    repo: string;    // "claude-code-action"
  };
  actor: string;     // Username who triggered
  issue?: {
    number: number;
    title: string;
    body: string;
  };
  comment?: {
    body: string;    // "@claude please review this"
    id: number;
  };
  pullRequest?: {
    number: number;
    diff: string;
  };
}
```

**Claude Prompt Structure:**
```typescript
interface PromptConfig {
  path: string;           // Path to prompt file
  systemPrompt?: string;  // Instructions for Claude
  userMessage: string;    // The actual request
  context: {
    prDiff?: string;
    issueBody?: string;
    files?: string[];
  };
}
```

---

## 7. Configuration Files Explained

### 7.1 action.yml

This is the **main entry point** for the GitHub Action. It defines:
- **Inputs:** Parameters users can configure
- **Outputs:** Values the action produces
- **Runs:** The steps to execute

```yaml
# action.yml structure (simplified)
name: "Claude Code Action"
description: "AI assistant for GitHub"

inputs:
  anthropic_api_key:
    description: "API key for Claude"
    required: false
  trigger_phrase:
    description: "Phrase that triggers Claude"
    default: "@claude"

outputs:
  branch_name:
    description: "Branch created by Claude"

runs:
  using: "composite"        # Multi-step action
  steps:
    - name: Install Bun
      uses: oven-sh/setup-bun@v2

    - name: Prepare
      run: bun run src/entrypoints/prepare.ts

    - name: Run Claude
      run: bun run base-action/src/index.ts
```

**Traditional equivalent:** This is like a `Makefile` or Ant `build.xml` that orchestrates multiple steps.

### 7.2 tsconfig.json

TypeScript compiler configuration:

```json
{
  "compilerOptions": {
    "target": "ES2022",           // Output JS version
    "module": "ESNext",           // Module system
    "moduleResolution": "bundler", // How to find imports
    "strict": true,               // Enable all strict checks
    "noUnusedLocals": true,       // Error on unused variables
    "types": ["bun"]              // Include Bun type definitions
  }
}
```

**Traditional equivalent:** This is like `javac` compiler flags or a `CMakeLists.txt` configuration.

### 7.3 Workflow Files (.github/workflows/*.yml)

These define when and how the action runs:

```yaml
# .github/workflows/claude.yml
name: Claude Code Action

on:                          # Trigger conditions
  issue_comment:
    types: [created]
  pull_request_review:
    types: [submitted]

jobs:
  claude:
    runs-on: ubuntu-latest   # VM type

    steps:
      - uses: actions/checkout@v4
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

---

## 8. Comparing to Traditional Development

### 8.1 Paradigm Differences

| Aspect | C/C++/Java | This Codebase |
|--------|------------|---------------|
| **Execution** | Compiled ahead of time | Interpreted/JIT at runtime |
| **Type System** | Static (compile-time) | Static (TypeScript) + Runtime validation (Zod) |
| **Concurrency** | Threads, locks | Async/await, event loop |
| **Memory** | Manual (C/C++) or GC (Java) | Garbage collected |
| **Dependencies** | Static/dynamic linking | npm packages |
| **Entry Point** | `main()` function | `if (import.meta.main)` block |

### 8.2 Code Pattern Comparisons

**Error Handling:**
```typescript
// TypeScript (this codebase)
try {
  const result = await fetchData();
} catch (error) {
  if (error instanceof Error) {
    console.error(error.message);
  }
}

// Java equivalent
try {
    Result result = fetchData().get();
} catch (ExecutionException e) {
    System.err.println(e.getCause().getMessage());
}
```

**Null Safety:**
```typescript
// TypeScript - Optional chaining
const name = user?.profile?.name ?? "Unknown";

// Java equivalent (pre-Optional)
String name = "Unknown";
if (user != null && user.getProfile() != null) {
    name = user.getProfile().getName();
}
```

**Interface Definition:**
```typescript
// TypeScript interface
interface GitHubContext {
  eventName: string;
  repository: Repository;
  actor: string;
}

// Java equivalent
public interface GitHubContext {
    String getEventName();
    Repository getRepository();
    String getActor();
}
```

### 8.3 Build Process Comparison

```
C++ Project                    This TypeScript Project
───────────                    ───────────────────────
cmake ..                       bun install
make                           (no build step - Bun runs .ts directly)
./my_program                   bun run src/entrypoints/prepare.ts

Java Project                   This TypeScript Project
────────────                   ───────────────────────
mvn install                    bun install
mvn compile                    (no compile step)
java -jar target/app.jar       bun run src/index.ts
mvn test                       bun test
```

---

## 9. Building and Running

### 9.1 Prerequisites

1. **Install Bun:**
   ```bash
   # macOS/Linux
   curl -fsSL https://bun.sh/install | bash

   # Windows
   powershell -c "irm bun.sh/install.ps1 | iex"
   ```

2. **Clone and install:**
   ```bash
   git clone https://github.com/anthropics/claude-code-action.git
   cd claude-code-action
   bun install
   ```

### 9.2 Running Tests

```bash
# Run all tests
bun test

# Run specific test file
bun test test/sanitizer.test.ts

# Type check without running
bun run typecheck

# Format code
bun run format
```

### 9.3 Local Development

The action is designed to run in GitHub's infrastructure, but you can test components locally:

```bash
# Run type checking
bun run typecheck

# Run a specific script
bun run src/entrypoints/prepare.ts

# The base-action has its own test script
cd base-action
./test-local.sh
```

---

## 10. Glossary

| Term | Definition |
|------|------------|
| **API** | Application Programming Interface - a way for programs to communicate |
| **Async/Await** | JavaScript pattern for handling asynchronous operations |
| **Bun** | A fast JavaScript runtime (alternative to Node.js) |
| **CI/CD** | Continuous Integration/Continuous Deployment |
| **Claude** | An AI assistant created by Anthropic |
| **Composite Action** | A GitHub Action made of multiple steps |
| **GraphQL** | Query language for APIs (used for GitHub) |
| **JSON** | JavaScript Object Notation - a data format |
| **JWT** | JSON Web Token - a secure token format |
| **LLM** | Large Language Model (the AI technology behind Claude) |
| **MCP** | Model Context Protocol - standard for AI tool integration |
| **Node.js** | JavaScript runtime for server-side code |
| **npm** | Node Package Manager |
| **OIDC** | OpenID Connect - authentication protocol |
| **Octokit** | GitHub's official API client library |
| **Promise** | JavaScript object representing async operation result |
| **REST** | Representational State Transfer - API architecture style |
| **Runtime** | The environment that executes code |
| **TypeScript** | JavaScript with static types |
| **Webhook** | HTTP callback triggered by an event |
| **YAML** | Yet Another Markup Language - config file format |
| **Zod** | TypeScript library for runtime type validation |

---

## Summary

This codebase implements a GitHub Action that:

1. **Listens** for GitHub events (comments, PRs, issues)
2. **Validates** permissions and trigger conditions
3. **Prepares** context for Claude (code, diffs, conversation)
4. **Executes** Claude with MCP tools for GitHub access
5. **Processes** results (creates PRs, posts comments)

The architecture uses:
- **TypeScript** for type-safe code
- **Bun** for fast execution
- **GitHub Actions** for automation
- **MCP** for AI-to-tool communication
- **Anthropic API** for AI capabilities

For traditional developers, the key mental shifts are:
- Async/await instead of threads
- JSON/YAML instead of XML/properties
- npm packages instead of static linking
- Event-driven instead of procedural flow

---

*This documentation was created to help traditional developers understand modern full-stack development patterns in the context of AI-assisted programming.*
