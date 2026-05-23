# AI Agent Skills Workflow

A structured, phase-gated workflow for producing optimal software engineering output using AI coding agents. Based on the 5 core skills from [aihero.dev](https://www.aihero.dev/5-agent-skills-i-use-every-day).

---

## Table of Contents

1. [What Is This Workflow?](#what-is-this-workflow)
2. [Why Should I Use It?](#why-should-i-use-it)
3. [Installation & Setup](#installation--setup)
   - [Claude Code Setup](#claude-code-setup)
   - [Codex Setup](#codex-setup)
4. [How Should I Use It?](#how-should-i-use-it)
   - [Starting from an Empty Folder](#scenario-1-empty-folder--greenfield-project)
   - [Working on an Existing Codebase](#scenario-2-existing-codebase)
   - [Working Across Multiple Codebases](#scenario-3-multiple-existing-codebases)
5. [Benefits and Drawbacks](#benefits-and-drawbacks)
6. [Version A: Claude Code](#version-a-claude-code)
7. [Version B: Codex (OpenAI)](#version-b-codex-openai)

---

## What Is This Workflow?

This workflow is a **phase-gated development process** that sits in front of AI coding agents to prevent the most common failure mode: an agent that writes code confidently but incorrectly because the problem was never properly defined.

It is built around five sequential skills, each acting as a gate before the next phase begins:

```
[Architecture Review] → [Discovery] → [Definition] → [Planning] → [Execution]
      /improve               /grill-me       /to-prd        /to-issues      /tdd
```

Each skill is a prompt-driven agent instruction that guides the AI through a specific discipline:

| Skill | Phase | What It Does |
|---|---|---|
| `/improve-codebase-architecture` | Baseline | Analyzes the codebase for agent-hostile patterns and fixes them |
| `/grill-me` | Discovery | Interviews you relentlessly until the problem is fully understood |
| `/to-prd` | Definition | Converts the discussion into a structured Product Requirements Document |
| `/to-issues` | Planning | Decomposes the PRD into vertical slices as independently executable tasks |
| `/tdd` | Execution | Enforces test-driven development (red → green → refactor) per task |

The workflow is not a rigid checklist — it is a discipline. Phases can be revisited, and some may be lighter depending on the size and nature of the task. But **no phase should be skipped entirely**.

---

## Why Should I Use It?

### The Core Problem This Solves

AI coding agents fail predictably in three ways:

1. **Misaligned implementation** — the agent builds something technically correct but functionally wrong, because it inferred intent rather than receiving a spec.
2. **Untestable output** — the agent produces tightly coupled code that cannot be verified, leading to silent bugs in production.
3. **Architectural decay** — each agent session adds complexity to a codebase that was already hard to navigate, compounding the problem over time.

This workflow eliminates all three failure modes by ensuring the agent always operates within a well-defined problem, against a testable interface, in a navigable codebase.

### The Underlying Principles

**Garbage in = garbage out.** An agent is a force multiplier. If the input (the codebase, the spec, the task definition) is low quality, the agent will produce low-quality output faster. The phases before execution exist entirely to raise input quality.

**Frederick P. Brooks' Design Tree.** Good design requires walking every branch of a decision tree — exploring options, resolving dependencies, and making explicit choices — before committing to a path. The `/grill-me` skill operationalizes this.

**Vertical slices over horizontal layers.** Tasks decomposed as "do all the database work, then all the API work, then all the UI work" create blocking dependencies and integration risk. Vertical slices (thin cuts that touch all layers to deliver one behavior) enable parallel work and produce testable increments.

**TDD as consistency insurance.** Test-driven development is the single most reliable technique for making agent output consistent. Tests define the contract; the agent fills the implementation. Without tests written first, the agent decides the contract implicitly — and often incorrectly.

---

## Installation & Setup

### Claude Code Setup

**Prerequisites**
- Claude Code CLI installed. If not yet installed:
  ```bash
  npm install -g @anthropic-ai/claude-code
  ```
- A Claude account with an active plan (Pro or higher recommended for long sessions)
- Git installed and configured

**Step 1 — Clone the skills repository**

The 5 skills are maintained by Matt Pocock in a public repository. Clone it somewhere permanent on your machine (outside your project folders):

```bash
git clone https://github.com/mattpocock/skills.git ~/claude-skills
```

**Step 2 — Register the skills with Claude Code**

Claude Code loads custom slash commands from a `.claude/commands/` directory. You can register the skills at the global level (available in every project) or at the project level (available only in one repo).

**Option A — Global (recommended): available in all projects**

```bash
mkdir -p ~/.claude/commands
cp ~/claude-skills/grill-me.md ~/.claude/commands/grill-me.md
cp ~/claude-skills/to-prd.md ~/.claude/commands/to-prd.md
cp ~/claude-skills/to-issues.md ~/.claude/commands/to-issues.md
cp ~/claude-skills/tdd.md ~/.claude/commands/tdd.md
cp ~/claude-skills/improve-codebase-architecture.md ~/.claude/commands/improve-codebase-architecture.md
```

**Option B — Project-level: available only in the current repo**

Run this from your project root:

```bash
mkdir -p .claude/commands
cp ~/claude-skills/grill-me.md .claude/commands/grill-me.md
cp ~/claude-skills/to-prd.md .claude/commands/to-prd.md
cp ~/claude-skills/to-issues.md .claude/commands/to-issues.md
cp ~/claude-skills/tdd.md .claude/commands/tdd.md
cp ~/claude-skills/improve-codebase-architecture.md .claude/commands/improve-codebase-architecture.md
```

**Step 3 — Verify the skills are registered**

Start a Claude Code session in any project:

```bash
claude
```

Type `/` in the prompt and confirm that the following commands appear in the autocomplete list:
- `/grill-me`
- `/to-prd`
- `/to-issues`
- `/tdd`
- `/improve-codebase-architecture`

**Step 4 — Create a CLAUDE.md file in your project (optional but recommended)**

`CLAUDE.md` is read by Claude Code at the start of every session. Use it to persist project-level context so you do not have to re-explain conventions each time.

Create it at your project root:

```bash
touch CLAUDE.md
```

Add content like:

```markdown
# Project Context

## Tech Stack
- Language: TypeScript
- Framework: Next.js 14
- Test runner: Vitest
- Package manager: pnpm

## Commands
- Run tests: `pnpm test`
- Start dev server: `pnpm dev`
- Lint: `pnpm lint`

## Conventions
- All new modules must have a corresponding test file
- PRDs live in `docs/prd-*.md`
- Task lists live in `docs/tasks-*.md`
```

**Step 5 — Install the GitHub CLI (optional, for `/to-issues`)**

The `/to-issues` skill can create GitHub Issues directly if the `gh` CLI is available:

```bash
# macOS
brew install gh

# Authenticate
gh auth login
```

---

### Codex Setup

**Prerequisites**
- An OpenAI account with access to Codex or ChatGPT (GPT-4o or higher recommended)
- Optionally: the OpenAI Codex CLI if using terminal-based access

**Step 1 — Install the Codex CLI (terminal access)**

```bash
npm install -g @openai/codex
```

Authenticate with your OpenAI API key:

```bash
export OPENAI_API_KEY=your_api_key_here
```

Add the export to your shell profile to persist it:

```bash
echo 'export OPENAI_API_KEY=your_api_key_here' >> ~/.zshrc
source ~/.zshrc
```

**Step 2 — Create a skills folder for your system prompts**

Since Codex does not have native slash commands, skills are stored as plain text files you paste into conversations:

```bash
mkdir -p ~/codex-skills
```

**Step 3 — Create a skill file for each phase**

Create the following files. You can copy the content from the system prompts defined in [Version B: Codex](#version-b-codex-openai) below.

```bash
touch ~/codex-skills/grill-me.md
touch ~/codex-skills/to-prd.md
touch ~/codex-skills/to-issues.md
touch ~/codex-skills/tdd.md
touch ~/codex-skills/improve-codebase-architecture.md
```

Fill each file with the corresponding system prompt from the Codex section of this document. Once populated, your usage workflow is:

```bash
# Print a skill prompt so you can copy-paste it into a conversation
cat ~/codex-skills/grill-me.md
```

**Step 4 — Create a Custom GPT (optional, for ChatGPT users)**

If you use ChatGPT rather than the Codex CLI, you can embed the skill prompts into a Custom GPT so they are always available without copy-pasting:

1. Go to [chatgpt.com](https://chatgpt.com) → **Explore GPTs** → **Create**
2. In the **Instructions** field, paste the system prompts for all 5 skills, separated by headers
3. Name it "AI Agent Workflow Assistant"
4. Save and use it as your default GPT for all development sessions

**Step 5 — Create a session anchor template**

Because Codex does not retain state between conversations, create a reusable anchor template you paste at the start of every session:

```bash
cat > ~/codex-skills/session-anchor.md << 'EOF'
## Session Context

- Project: [project name]
- Repository: [repo path or URL]
- Feature in progress: [feature name]
- Current phase: [Discovery / Definition / Planning / Execution]
- Current task: [issue title or description]
- PRD location: [paste inline or reference file]
- Files in scope this session: [list relevant files]
EOF
```

Fill in the brackets at the start of each session before pasting any skill prompt.

---

### Keeping Skills Up to Date

Matt Pocock updates the skills repository periodically. To pull the latest versions:

```bash
cd ~/claude-skills && git pull

# Re-copy updated files to Claude Code commands (if using global install)
cp ~/claude-skills/grill-me.md ~/.claude/commands/grill-me.md
cp ~/claude-skills/to-prd.md ~/.claude/commands/to-prd.md
cp ~/claude-skills/to-issues.md ~/.claude/commands/to-issues.md
cp ~/claude-skills/tdd.md ~/.claude/commands/tdd.md
cp ~/claude-skills/improve-codebase-architecture.md ~/.claude/commands/improve-codebase-architecture.md
```

---

## How Should I Use It?

### Scenario 1: Empty Folder / Greenfield Project

You have no code. You have an idea. This is the highest-risk scenario for agents because there is no existing structure to anchor the output.

**Step 1 — Discovery**

Start immediately with `/grill-me`. Do not scaffold anything first. Describe your idea at a high level and let the agent interview you through every branch of the design tree. Expect 20-50+ questions covering:
- User personas and their goals
- Core behaviors and edge cases
- Data model and state management
- Integration points and external dependencies
- Non-functional requirements (performance, security, scalability)
- What the product explicitly does NOT do

Do not move forward until you and the agent have reached a shared, documented understanding.

**Step 2 — Definition**

Run `/to-prd`. The agent will explore your answers and produce a structured PRD including:
- Problem statement
- User stories (in Agile format: "As a [persona], I want [behavior] so that [outcome]")
- Acceptance criteria per user story
- Out-of-scope items (explicit)
- Module sketch (high-level structure)

Commit this PRD to version control before writing any code. It is your single source of truth.

**Step 3 — Planning**

Run `/to-issues`. The agent reads the PRD and produces a set of vertical slice tasks, each one:
- Self-contained and independently executable
- Cutting through all layers (data → logic → interface)
- Tagged with explicit blocking relationships

Review the issue list. Reorder, merge, or split as needed. The output should read like a Kanban board where any unblocked card can be picked up by an agent with no additional context.

**Step 4 — Execution**

For each issue, run `/tdd`. The agent will:
1. Confirm the interface contract for the module
2. Write a failing test (red)
3. Write the minimum implementation to pass the test (green)
4. Identify and apply refactoring opportunities (refactor)

Never mark a task done without a passing test.

**Step 5 — Architecture Review (after first working increment)**

Once you have a working vertical slice, run `/improve-codebase-architecture`. This is the earliest point at which architectural patterns are visible and correctable. Repeat weekly or after every development surge.

---

### Scenario 2: Existing Codebase

You have a running project. You want to add a feature, fix a significant bug, or refactor a subsystem.

**Step 0 — Architecture Review First**

Before touching anything, run `/improve-codebase-architecture`. This produces a map of:
- Files that are hard to understand in isolation
- Functions claiming to be pure but masking integration side effects
- Tightly coupled modules that will make new work risky

Do not skip this step on an existing codebase. The architecture review often reveals that the planned feature needs to be approached differently than initially assumed.

**Step 1 — Discovery (Scoped)**

Run `/grill-me` scoped to the feature or change. In your initial description, include:
- Which part of the codebase is affected
- What the current behavior is
- What the desired behavior is
- Known constraints (backward compatibility, performance budgets, etc.)

The agent's questions will be informed by the existing architecture. This is more focused than the greenfield version but equally important.

**Step 2 — Definition**

Run `/to-prd`. Because a codebase already exists, the PRD here is lighter but must still include:
- Explicit change scope (what modules are affected)
- User stories for the new or changed behavior
- Migration notes if existing behavior changes
- What must not break (regression scope)

**Step 3 — Planning**

Run `/to-issues`. The agent will map the PRD changes to the existing file structure. Vertical slices here often look like:
- Slice 1: Extend data layer + write integration test
- Slice 2: Add business logic + write unit test
- Slice 3: Expose via API + write contract test
- Slice 4: Update UI / CLI / consumer + write end-to-end test

**Step 4 — Execution**

Run `/tdd` per issue. On an existing codebase, the agent should also run existing tests before and after each slice to catch regressions before they accumulate.

---

### Scenario 3: Multiple Existing Codebases

You are working across a system of services, repositories, or packages (monorepo or polyrepo). This is the highest-complexity scenario and requires explicit coordination.

**Step 0 — Architecture Review Per Repo**

Run `/improve-codebase-architecture` in each affected repository. Build a cross-repo dependency map. Identify which repos are upstream (produce contracts) and which are downstream (consume contracts). Changes must flow upstream-to-downstream.

**Step 1 — Discovery (System-Level)**

Run `/grill-me` with explicit system scope. Your initial description must include:
- All repositories involved
- The integration points between them (APIs, events, shared libraries)
- Which team or agent owns each repo
- The deployment topology (are these released independently?)

The agent will surface cross-cutting concerns: shared schema changes, API versioning, consumer compatibility, and rollout sequencing.

**Step 2 — Definition (One PRD Per Boundary)**

Run `/to-prd` once for the system-level change, then once per repository boundary. Each repo-level PRD must include:
- Its contract with upstream repos (what it receives)
- Its contract with downstream repos (what it produces)
- Its internal changes

This creates a dependency graph of PRDs that mirrors the dependency graph of the repositories.

**Step 3 — Planning (Sequenced by Dependency)**

Run `/to-issues` per repository. The blocking relationships must reflect cross-repo dependencies:
- Upstream repo issues must be completed (and deployed or published) before downstream issues begin
- Shared library changes must be versioned and released before consumers update

Tag each issue with its repo identifier. If using a project management tool, maintain a single board with swim lanes per repo.

**Step 4 — Execution (Upstream First)**

Run `/tdd` starting with the upstream-most repository. Do not begin downstream work until upstream contracts are stable (tests passing, interfaces finalized). Changing an upstream interface after downstream work has started is the primary source of cross-repo integration bugs.

**Coordination Protocol**

In a multi-repo context, each agent session should begin with a brief context injection:
- Which repo is in scope for this session
- The system PRD reference
- The current state of upstream contracts (what is locked vs. in-flight)

This prevents an agent from making assumptions about interfaces it cannot see.

---

## Benefits and Drawbacks

### Benefits

| Benefit | Detail |
|---|---|
| Eliminates misaligned output | The agent never guesses intent — every implementation follows a written spec |
| Reduces rework | Ambiguity caught in discovery costs minutes; ambiguity caught in production costs days |
| Makes output testable by default | TDD enforces interface contracts, making agent output verifiable |
| Enables parallel agent work | Vertical slices with declared dependencies allow multiple agents to work simultaneously |
| Produces durable artifacts | PRDs and issues remain useful after implementation as documentation and regression anchors |
| Compounds over time | Each architecture review improves the codebase for every future agent session |
| Model-agnostic | The discipline works regardless of which AI agent executes it |

### Drawbacks

| Drawback | Detail |
|---|---|
| Upfront time cost | The discovery and definition phases add time before any code is written — frustrating for small changes |
| Overhead for trivial tasks | A one-line bug fix does not need a PRD; applying the full workflow to every change creates friction |
| Requires good interviewing | The quality of `/grill-me` output depends on how well you describe the problem initially |
| PRD maintenance burden | On a fast-moving project, keeping PRDs up to date as requirements change requires discipline |
| TDD skill requirement | Effective test-driven development requires understanding of testable design; the agent cannot compensate for a codebase with no test infrastructure |
| Architecture review is periodic, not continuous | The `/improve-codebase-architecture` skill is a snapshot; architectural decay can still accumulate between runs |

### When to Apply the Full Workflow vs. a Subset

| Situation | Recommended Phases |
|---|---|
| New product or major feature | All 5 phases |
| Medium feature in existing codebase | Architecture review + Discovery + Definition + TDD (skip issues for solo work) |
| Small feature or enhancement | Discovery + TDD |
| Bug fix (known root cause) | TDD only |
| Bug fix (unknown root cause) | Discovery + TDD |
| Refactoring | Architecture review + TDD |
| Hotfix under time pressure | TDD only, then retroactive discovery |

---

## Version A: Claude Code

Claude Code is a terminal-native, session-based agent with first-class support for slash commands (skills), file system access, and long-context retention within a session.

### Installing the Skills

The skills in this workflow are sourced from [mattpocock/skills](https://github.com/mattpocock/skills). Clone or reference this repository, then register each skill in your `.claude/` directory or invoke them directly as slash commands if your Claude Code version supports remote skill loading.

```
/grill-me
/to-prd
/to-issues
/tdd
/improve-codebase-architecture
```

### Claude Code-Specific Workflow

#### Phase 0 — Architecture Baseline

```
/improve-codebase-architecture
```

Claude Code will traverse the repository using its file system tools, read source files, and produce a structured analysis with actionable recommendations. It will distinguish between:
- **Deep modules** (preferred): wide interface, complex internals, easy to test
- **Shallow modules** (problematic): narrow interface wrapping simple internals, adds coupling without adding abstraction

Instruct Claude to make the recommended structural changes before proceeding. Commit these changes independently of any feature work.

#### Phase 1 — Discovery

```
/grill-me
```

Describe your feature or change in 2-5 sentences. Claude will enter interview mode and ask questions one branch at a time, waiting for your answers before proceeding. The session is conversational — answer thoroughly, and allow Claude to push back if your answers are contradictory or incomplete.

**Claude Code tips:**
- Claude retains full conversation context within a session. Do not start a new session between `/grill-me` and `/to-prd` — carry them forward in the same conversation.
- If a question reveals a blocker or dependency you had not considered, stop and resolve it before continuing the interview.
- You can ask Claude to summarize the current shared understanding at any point with: "Summarize what we've aligned on so far."

#### Phase 2 — Definition

```
/to-prd
```

Claude will use the `/grill-me` conversation as context and produce a PRD document. By default it will write this to a file (e.g., `docs/prd-<feature-name>.md`). Review the document carefully — it is the artifact that anchors all subsequent work.

**Claude Code tips:**
- If the PRD misses something surfaced in the interview, ask Claude to update specific sections rather than regenerating. Claude Code's Edit tool makes surgical changes.
- Add the PRD to version control immediately: `git add docs/prd-*.md && git commit -m "docs: add PRD for <feature>"`.
- If the project uses GitHub Issues, Claude Code can submit the PRD as an issue via the `gh` CLI: ask Claude to do this after the PRD is committed.

#### Phase 3 — Planning

```
/to-issues
```

Claude will read the PRD and the current codebase structure, then produce a set of GitHub Issues (or a markdown task list if not using GitHub). Each issue will be a vertical slice with:
- A title describing the behavior delivered
- Acceptance criteria pulled from the PRD
- A "Blocked by" field listing prerequisite issues

**Claude Code tips:**
- Claude Code can create GitHub Issues directly using `gh issue create`. Ask Claude to create them one by one so you can review and approve each before creation.
- After issues are created, ask Claude to print the blocking dependency tree so you can identify the critical path.
- For solo work, you can skip GitHub Issues and ask for a plain markdown task list in `docs/tasks-<feature>.md` instead.

#### Phase 4 — Execution

For each issue or task:

```
/tdd
```

Claude will confirm the interface (function signatures, API contract, event schema), write a failing test, implement the minimum code to pass, then propose refactors.

**Claude Code tips:**
- Always start a `/tdd` session by pasting or referencing the specific issue. Example: "Pick up issue #12: Add user authentication endpoint."
- After each red → green cycle, run your test suite before asking Claude to proceed: `npm test` or equivalent. Claude Code can run this itself if given permission.
- Use Plan Mode (`/plan`) before execution on any issue touching more than 3 files. This lets you review Claude's intended approach before any files are changed.
- After each issue is complete, commit immediately. Small, issue-scoped commits make rollback trivial.

#### Recurring Maintenance

Schedule `/improve-codebase-architecture` weekly or after every significant development burst. In Claude Code, you can use the `/schedule` skill to automate this reminder, or run it manually at the start of each week's work session.

---

### Claude Code Session Management

Claude Code maintains context within a session but resets between sessions. To preserve continuity across sessions:

1. Always commit the PRD and task list to version control — these are your persistent context anchors.
2. Begin each new session with: "Read `docs/prd-<feature>.md` and `docs/tasks-<feature>.md`, then tell me the current state of work."
3. Use `CLAUDE.md` in the project root to store persistent project-level instructions that Claude Code reads at session start (coding conventions, test commands, architecture constraints).

---

## Version B: Codex (OpenAI)

Codex (as accessed via the OpenAI Codex CLI or the Codex agent in ChatGPT) operates differently from Claude Code in key ways: it is more stateless between turns, relies more on explicit file context injection, and its skill system is not slash-command-based. The same 5-phase discipline applies, but the mechanics differ.

### Key Differences from Claude Code

| Dimension | Claude Code | Codex |
|---|---|---|
| Skill invocation | Slash commands (`/grill-me`) | Custom GPT instructions or system prompts |
| Context retention | Long within a session | Shorter; benefits from explicit file injection |
| File system access | Native, deep | Sandboxed; requires explicit upload or API access |
| Session persistence | Moderate | Low; must re-anchor each session |
| GitHub integration | Via `gh` CLI natively | Via plugins or manual copy-paste |
| Test execution | Native terminal execution | Requires Code Interpreter or external runner |

### Setting Up the Skills in Codex

Since Codex does not have native slash commands, each skill is implemented as a **system prompt segment** or a **custom instruction block** that you paste at the start of a conversation or embed in a Custom GPT.

Below are the system prompt equivalents for each skill:

---

**Skill: Discovery (`/grill-me` equivalent)**

Paste this at the start of a conversation before describing your feature:

```
You are in interview mode. Your goal is to reach a complete shared understanding of the feature I am about to describe before any implementation is discussed. Interview me relentlessly about every aspect of the plan. Walk down each branch of the design tree, resolving dependencies between decisions one by one. Ask one or two questions at a time. Do not suggest implementations. Do not write code. Continue until you can summarize the feature without any ambiguity remaining.
```

---

**Skill: Definition (`/to-prd` equivalent)**

After the discovery conversation, paste:

```
Based on our conversation, produce a Product Requirements Document with the following sections:
1. Problem Statement
2. User Personas
3. User Stories (format: "As a [persona], I want [behavior] so that [outcome]")
4. Acceptance Criteria (per user story)
5. Out-of-Scope Items
6. Module Sketch (high-level component diagram in plain text)
7. Open Questions (anything still unresolved)

Write this as a complete markdown document suitable for committing to version control.
```

---

**Skill: Planning (`/to-issues` equivalent)**

After the PRD is produced, paste the PRD content and then:

```
Based on this PRD and the codebase structure I will describe, decompose the work into independently executable vertical slices. Each slice must:
- Deliver a complete, testable behavior (touching data, logic, and interface layers)
- Include a title, description, acceptance criteria, and a "Blocked by" field
- Be executable without requiring completion of unblocked sibling slices

Produce the output as a numbered list of GitHub Issue drafts in markdown format.
```

Then describe or paste the relevant parts of the codebase structure.

---

**Skill: Execution (`/tdd` equivalent)**

For each issue, start a new conversation and paste:

```
You are implementing a feature using strict test-driven development. Follow this loop exactly:
1. Confirm the interface contract (function signatures, API schema, event format) with me before writing anything.
2. Write a single failing test that covers the most important behavior.
3. Write the minimum implementation code to make that test pass.
4. Identify refactoring opportunities and apply them if they do not require new tests.
5. Repeat from step 2 for the next behavior.

Do not write implementation code before a test exists for it. Do not write multiple tests at once.
```

Then paste the relevant issue description and any existing code context.

---

**Skill: Architecture Review (`/improve-codebase-architecture` equivalent)**

Paste representative files or a directory structure, then:

```
Analyze this codebase for architectural patterns that will make AI-assisted development harder:
1. Files that are difficult to understand without reading multiple other files
2. Functions that claim to be pure but have hidden side effects or integrations
3. Tightly coupled modules where a change in one requires changes in many others
4. Shallow modules (narrow interfaces wrapping trivial logic) that could be deepened

For each issue found, explain the problem, the risk it creates, and a specific refactoring recommendation. Prioritize by impact on testability and agent navigability.
```

---

### Codex-Specific Workflow

#### Phase 0 — Architecture Review

Because Codex has limited file system access, you must provide context explicitly. Use one of these methods:
- Paste the directory tree (`find . -type f | head -100`)
- Upload a zip of the relevant source files if using ChatGPT with file upload
- Use the Codex API with file attachments if using the API directly

Run the architecture review prompt and apply recommendations manually (Codex cannot edit files in your local environment without the Codex CLI).

#### Phase 1 — Discovery

Start a fresh conversation. Paste the discovery system prompt, then describe your feature. Answer every question thoroughly. When the interview is complete, ask Codex to produce a summary of shared understanding. Copy this summary — it is your input for the next phase.

**Codex tips:**
- Codex's context window is shorter than Claude's. If the interview runs long, ask Codex to produce an interim summary every 10 questions and use that summary as the starting context for continuation.
- Do not use the same conversation for discovery and definition if the discovery session was long — start fresh with the summary as context.

#### Phase 2 — Definition

Start a new conversation. Paste the discovery summary, then paste the PRD system prompt. Review the output carefully. Codex may hallucinate specifics (file names, library APIs) that were not discussed — verify these against your actual codebase before committing.

**Codex tips:**
- Always ask: "What assumptions did you make that were not explicitly discussed in our interview?" This surfaces hallucinated specifics before they propagate into the issues.
- Save the PRD locally immediately. Codex does not persist artifacts between conversations.

#### Phase 3 — Planning

Start a new conversation. Paste:
1. The PRD (full text)
2. Your directory structure or relevant file list
3. The planning system prompt

Review each issue draft for accuracy. Codex may create horizontal slices by default (all DB work first, etc.) — push back explicitly: "Reorganize these as vertical slices where each slice delivers a complete testable behavior."

#### Phase 4 — Execution

For each issue, start a new conversation. Paste:
1. The TDD system prompt
2. The issue description
3. The relevant source files for this slice (only the files the agent will touch)

**Codex tips:**
- Paste only the files relevant to the current slice. Over-providing context leads Codex to make changes in files it should not touch.
- After each red → green cycle, copy the generated code back to your local environment, run your test suite, and confirm the test passes before continuing.
- If Codex drifts from TDD (writes implementation before a test), interrupt it: "Stop. Write the test first. Do not write implementation until I confirm the test is correct."

#### Session Re-anchoring

Because Codex does not retain state, begin every session with an explicit anchor block:

```
Context for this session:
- Project: [project name]
- Feature in progress: [feature name]
- PRD: [paste or attach]
- Current task: [issue title and description]
- Files in scope: [list or attach]
```

This replaces the persistent session context that Claude Code provides natively.

---

## Workflow Comparison Summary

| Dimension | Claude Code | Codex |
|---|---|---|
| Skill invocation | Native slash commands | Manual system prompt injection |
| Context continuity | Automatic within session | Manual re-anchoring per session |
| File editing | Native, surgical | Copy-paste or API-based |
| Test execution | Native terminal | Manual copy-paste to local environment |
| GitHub integration | Native via `gh` CLI | Manual or via plugins |
| PRD persistence | Committed to repo, re-read by agent | Manual paste at session start |
| Best for | Long sessions, complex features, multi-file refactors | Focused single-task sessions, API-driven workflows |
| Biggest risk | Session context loss between days | Hallucination of unstated specifics in PRD/issues |

---

## Quick Reference Card

```
GREENFIELD:            grill-me → to-prd → to-issues → tdd (→ improve-arch after first increment)
EXISTING CODEBASE:     improve-arch → grill-me → to-prd → tdd (→ to-issues if complex)
MULTIPLE CODEBASES:    improve-arch (each) → grill-me (system) → to-prd (per boundary) → to-issues (sequenced) → tdd (upstream first)
BUG FIX (known):       tdd
BUG FIX (unknown):     grill-me → tdd
REFACTOR:              improve-arch → tdd
```

---

*Workflow based on skills by Matt Pocock — [aihero.dev](https://www.aihero.dev/5-agent-skills-i-use-every-day) — skills repository: [mattpocock/skills](https://github.com/mattpocock/skills)*
