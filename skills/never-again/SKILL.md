---
name: never-again
description: >
  Root-cause a mistake and implement systemic repairs to prevent recurrence.
  Analyzes what went wrong, surveys all customization layers (instructions,
  skills, agents, memory, docs) for the gap that allowed it, recommends
  specific fixes, and implements them on approval.
  USE FOR: root cause, postmortem, why did that happen, prevent this mistake,
  continuous improvement, never again, what went wrong, fix the process.
  DO NOT USE FOR: fixing the immediate bug itself (do that first, then invoke
  this skill), code review, or drift audits.
argument-hint: >
  Describe the mistake, or say "from this session" to analyze the most recent
  failure in conversation history.
---

# Never Again

When a mistake is discovered — a wrong approach, a repeated error, a misunderstood convention, a flawed instruction — this skill performs root-cause analysis and implements systemic repairs across the project's customization layers so the same class of mistake doesn't recur.

**Philosophy:** Mistakes are inevitable. Repeating the same mistake is a system failure. Every error is an opportunity to strengthen the guardrails — not by adding bureaucracy, but by putting knowledge where it will be found at the right moment.

---

## When to Invoke

- After fixing a bug or reversing a wrong approach — "why did that happen?"
- When the user notices a pattern of similar mistakes across sessions
- When an agent followed bad instructions or missed good ones
- When documentation led to the wrong conclusion
- When a convention was violated because nothing enforced it

**Typical flow:** Fix the immediate problem first, *then* invoke this skill to prevent recurrence.

---

## Process

### 1. Identify the Mistake

Establish exactly what went wrong. Sources (check in order):

1. **User description** — the user may describe the mistake directly in their prompt
2. **Conversation history** — scan recent turns for error patterns, corrections, "no, that's wrong", backtracking, or retries
3. **Ask if unclear** — if neither source gives a clear picture, ask: "What was the mistake? What happened vs. what should have happened?"

Produce a concise **Mistake Statement**:

```
MISTAKE: {what happened}
EXPECTED: {what should have happened}
IMPACT: {what it cost — wasted time, wrong code, broken build, etc.}
```

**Checkpoint:** If the mistake and root cause are completely obvious (e.g., a known convention was violated), state the mistake statement and proceed directly to the repair phase. Otherwise, present the mistake statement to the user and confirm you've understood correctly before continuing. Don't waste time on deep analysis if the user needs to course-correct your understanding first.

### 2. Root-Cause Analysis

Ask "why?" iteratively (5-whys style) until you reach a systemic root cause — not "I made a mistake" but "what about the system allowed or encouraged this mistake?"

Root causes typically fall into these categories:

| Category | Example |
|----------|---------|
| **Missing instruction** | No rule existed to guide the correct behavior |
| **Wrong instruction** | A rule existed but was incorrect or outdated |
| **Buried instruction** | The rule exists but is in a file that wasn't loaded for this context |
| **Missing enforcement** | The rule is documented but nothing enforces it (no lint rule, no CI check, no validation) |
| **Stale documentation** | Docs describe how something *used to* work |
| **Skill/agent gap** | A skill or agent definition doesn't handle this case or gives wrong guidance |
| **Memory gap** | A lesson was learned before but never recorded in persistent memory |
| **Assumption error** | The agent assumed something that should have been verified |
| **Missing context** | The right information existed but wasn't available when the decision was made |

Identify which category (or categories) apply. Be specific about *which file* or *which gap*.

### 3. Survey Repair Targets

Systematically check each customization layer for where a fix would be most effective. **Read the actual files** — don't guess at their contents.

| Layer | Where to look | What to look for |
|-------|--------------|-----------------|
| **copilot-instructions.md** | `.github/copilot-instructions.md` | Should a new Common Mistakes row be added? Is an existing rule wrong or unclear? |
| **Instruction files** | `.github/instructions/*.instructions.md` | Is there an `applyTo`-scoped instruction file that should catch this? Does one exist but miss this case? |
| **Skills** | `.github/skills/*/SKILL.md` | Did a skill give wrong guidance? Does a skill need a new constraint or step? |
| **Agents** | `.github/agents/*.agent.md` | Did an agent definition miss a responsibility or give wrong context? |
| **User memory** | `/memories/*.md` | Should a lesson be recorded for cross-session persistence? Is an existing memory note wrong? |
| **Repo memory** | `/memories/repo/*.md` | Should a repo-scoped fact be recorded? |
| **Documentation** | Project docs (README, guides, architecture docs) | Is the relevant doc wrong, missing, or misleading? |
| **Lint/CI** | Lint configs, CI pipelines, validation scripts | Could this be enforced by tooling rather than relying on the agent to remember? |

For each layer, determine:
- **Not relevant** — this layer couldn't have prevented the mistake
- **Gap found** — this layer is missing something that would have helped
- **Wrong content** — this layer has content that contributed to the mistake
- **Sufficient** — this layer already covers this case (the mistake happened despite it)

### 4. Recommend Repairs

For each gap or wrong-content finding, propose a specific repair:

```markdown
## Repair Plan

### Repair 1: {short title}
- **Target**: {file path}
- **Type**: Add / Update / Remove
- **Rationale**: {why this repair prevents recurrence}
- **Change**: {description of what to add, change, or remove}
- **Priority**: Implement now / Defer

### Repair 2: ...
```

**Prioritization rules:**
- **Implement now** — The fix is small, clearly correct, and directly prevents the mistake class. Memory updates, Common Mistakes table additions, instruction file tweaks.
- **Defer** — The fix is large, requires design thought, or has side effects. Lint rules, CI changes, major doc rewrites. Note these as follow-up items.

**Quality criteria for repairs:**
- Each repair must be **specific** — name the file, the section, the exact change
- Each repair must be **minimal** — fix the gap, don't over-engineer
- Each repair must be **testable** — after implementation, describe how to verify it works
- Prefer repairs at the **point of use** — an instruction loaded via `applyTo` when editing the relevant file type is better than a general rule in copilot-instructions.md
- Prefer **enforceable** repairs over **advisory** ones — a lint rule beats a doc paragraph
- Don't add redundant guardrails — if the rule already exists somewhere, strengthen or relocate it rather than duplicating

**Choosing the right location for repairs:**

| Location | Audience | Use when |
|----------|----------|----------|
| `.github/instructions/*.instructions.md` with `applyTo` | All contributors working on matching files in this repo | The lesson is repo-specific and most useful when editing particular files. **This is the default choice for most repairs.** |
| `.github/copilot-instructions.md` | All AI conversations in this repo | The lesson is a universal guardrail or project-wide context. |
| `.github/skills/*/SKILL.md` or `.github/agents/*.agent.md` | AI agents following that skill/agent | The skill or agent itself gave wrong guidance or missed a step. |
| Project documentation (README, guides, etc.) | All contributors reading docs | The lesson is project knowledge, not an AI instruction. |
| `/memories/*.md` (user memory) | Only the current user, across all repos | **Last resort.** Only for purely personal preferences. Never use for project lessons — those must be version-controlled. |
| `/memories/repo/*.md` (repo memory) | Only the current user, in this repo | **Avoid.** Use only for personal scratch notes that don't belong in version control. |
| Lint rules / CI checks | All contributors, enforced automatically | The mistake can be detected mechanically. Best durability but highest cost to implement. |

**Key principle:** Repairs must be version-controlled so they help ALL contributors — not just the current user. If a lesson would prevent mistakes for other people or other AI sessions, it belongs in `.github/instructions/`, `.github/copilot-instructions.md`, skills, agents, or docs. **Never put project lessons in user memory or repo memory** — those files are invisible to the rest of the team.

### 5. Present and Confirm

Present the full analysis to the user:

```markdown
## Never Again: {short title}

### What Happened
{Mistake Statement from Step 1}

### Root Cause
{Analysis from Step 2}

### Repair Plan
{Repairs from Step 4, grouped by priority}

### Deferred Items
{Any larger changes noted for follow-up}
```

Then ask: **"Implement the repairs marked 'implement now'?"**

If the user says yes (or any affirmative), proceed to Step 6. If the user wants to adjust the plan, incorporate feedback and re-present.

### 6. Implement Repairs

Execute each "implement now" repair:

- **Memory updates**: Use the memory tool to create or update files
- **Instruction/skill/agent edits**: Edit the relevant files directly
- **copilot-instructions.md edits**: Be careful with size constraints if the project specifies a line budget
- **Documentation edits**: Follow the project's documentation guidelines

After each repair, briefly confirm what was changed.

### 7. Verify

After implementation, verify each repair:

- **Re-read modified files** to confirm the change is correct and doesn't break surrounding content
- **Check for errors** in any modified code files
- **Size check** copilot-instructions.md if it was modified

Report the verification results.

---

## Constraints

- **Don't fix the immediate bug.** This skill is for systemic repairs, not the bug itself. The bug should already be fixed (or the user explicitly says to analyze it pre-fix).
- **Don't add redundant rules.** If a rule already exists but was missed, the repair might be relocating it, not duplicating it.
- **Don't over-engineer.** A Common Mistakes row is often better than a new instruction file. Match the weight of the repair to the weight of the mistake.
- **Be honest about limits.** If the mistake was genuinely unforeseeable (not a pattern, not a convention violation), say so. Not every mistake needs a systemic fix. Sometimes the right answer is "this was a one-off judgment call."

---

## Examples

**Example 1: Agent used `console.log` in a project that bans it**
- Root cause: Missing enforcement — a style guide mentions it but the agent didn't check
- Repair: Verify ESLint rule catches it at build time (it does — so no code change needed, the agent just needs to run lint before declaring done). If no lint rule existed, the repair would be adding one.

**Example 2: Agent pushed to the wrong branch**
- Root cause: Branch was tracking the wrong upstream due to missing `--no-track` when created
- Repair: Add a Common Mistakes row to `.github/copilot-instructions.md` reinforcing the branching workflow, or add a constraint to the relevant shipping skill.

**Example 3: Documentation sent user to a moved file**
- Root cause: Stale documentation — a file was moved but the doc wasn't updated
- Repair: Fix the doc (implement now). Consider adding a validation script to catch broken internal references (defer — needs design).
