---
name: sweep
description: >
  Sweep the current chat session for escalations before archiving. Identifies
  incomplete work, deferred decisions, open questions, and untracked findings.
  USE FOR: sweep session, loose ends, before archive, what did I miss, session
  recap, session review, wrap up.
  DO NOT USE FOR: implementing the escalations (just reporting them), plan
  authoring, or code review.
---

# Sweep

Review the current conversation and identify everything that's unfinished or unresolved. Report it clearly so the user can decide what to do with each item.

---

## When to Invoke

- Before archiving a long or multi-topic session
- When the user says "sweep", "loose ends", "what did I miss", "wrap up", "before I archive"
- When the user feels overwhelmed by open threads across sessions

## Process

### 1. Scan the Conversation

Read through the full conversation history (it's in your context as the conversation summary + recent turns). Identify items in these categories:

| Category | What to look for |
|----------|-----------------|
| **Incomplete work** | Tasks discussed but not finished — steps planned but not implemented, "we'll come back to this", work that was started in conversation but abandoned mid-way |
| **Deferred decisions** | Design choices or questions raised but explicitly postponed — "let's decide later", "parking this for now" |
| **Open questions** | Questions asked (by user or agent) that never got answered |
| **Discovered issues** | Bugs, inconsistencies, or problems found during the session that weren't fixed |
| **Ideas & suggestions** | Feature ideas, improvement suggestions, or "we should also..." items that weren't acted on |

For each item, note:
- **What**: one-line description
- **Context**: enough detail to resume without re-reading the whole session (file paths, decisions made so far, why it was deferred)
- **Suggested next step**: what to do when picking this up

### 2. Check Session Memory

Read any existing files in `/memories/session/` — they may contain notes from earlier in the session that are themselves escalations or context for items found in step 1.

### 3. Produce the Sweep Report

Present the findings to the user as a structured summary in chat. Group by category. Use this format:

```
## Sweep

**Session topic(s):** {brief description of what the session covered}
**Date:** {today}

### Escalations

#### Incomplete Work
E1. **{title}** — {description}
   - Context: {relevant details, file paths}
   - Status: {verified still an issue / partially resolved / resolved — drop if resolved}
   - Recommendation: **{Do now | Defer | Drop}** — {why}
E2. ...

#### Deferred Decisions
E3. ...

#### Open Questions
E4. ...

#### Discovered Issues
E5. ...

#### Ideas & Suggestions
E6. ...

#### Plan Drift
E7. ...

### Verdict
{🟢 Clean session — no escalations}
{🟡 Has escalations — N items need attention}
{🔴 Significant open work — items listed above should be addressed or deliberately parked}
```

**Numbering rule:** Use `E` prefix with a continuous sequence across all categories (E1, E2, E3… not restarting for each heading). This gives every escalation a unique ID for easy reference in follow-up ("do E2 now, defer E4").

If a category has no items, omit it entirely.

### 4. Reality Check

Session history can be stale — something flagged as incomplete earlier may have been fixed in a later turn, or the codebase may have moved on from an issue discussed mid-session. Before presenting the report, **verify each item against the actual codebase**:

- Use `read_file`, `grep_search`, `list_dir`, or the Explore subagent to check whether the files, cross-references, or patterns mentioned still exist in the state described.
- If an item turns out to be resolved (file was deleted, link was fixed, entry point was added), **drop it from the report** — don't list things that aren't actually loose.
- If an item is partially resolved, note what's done and what remains.

**Scope rule:** Verification means confirming that items *found in conversation* still exist or were resolved. It does **not** mean scanning git status, uncommitted changes, or the file system for additional items that were never discussed. The sweep reports on the conversation, not the working tree.

### 5. Plan Drift Check

The session may have implemented work described by a plan document without updating the plan itself. This is a high-impact miss — a feature can ship while its tracking plan still says "draft" with all steps "Not started", making the work invisible to stakeholders.

**Procedure:**

1. **Identify the session's major work areas** — from the conversation summary, determine the key features, systems, or components that were built or significantly modified.
2. **Search for matching plans** — scan the workspace for plan files (commonly in a `plans/` or `docs/plans/` directory) whose title or summary relate to the session's work. Use `grep_search` or the Explore subagent.
3. **Check for drift** — for each matching plan, read its frontmatter (`status`) and implementation steps table. Flag as an escalation if:
   - The plan's `status` is `draft` or `ready` but the session implemented substantial work described by the plan
   - The plan has implementation steps marked `⬜ Not started` or `🔄 In progress` that are actually complete based on the session's work
   - The plan's design section describes interfaces or behavior that the implementation has diverged from
4. **Report any drift** — add plan-drift findings to the escalations list under a dedicated category. Each finding should name the plan file, its current status, and what's stale.

**Scope rule:** Only check plans that plausibly relate to the session's work — don't audit the entire plans directory. If the session was a small bug fix with no plan relationship, skip this step entirely.

### 6. Recommendations

For each surviving escalation, provide a concrete recommendation — not just "what to do" but **whether to do it at all**:

- **Do now** — Small, mechanical, or quick to address. **This is the default.** Most escalations from a session are small follow-ups (doc updates, missing test cases, consistency fixes) that take less time to do than to write down and remember. Bias heavily toward "Do now" — deferred small items accumulate into forgotten debt.
- **Defer** — Blocked on something else, requires significant design thought, or genuinely large. Park it deliberately with a reason.
- **Drop** — On reflection, not worth doing. The cost exceeds the benefit. Say so plainly.

**Bias rule:** If an item would take less than ~10 minutes to implement, it's "Do now" — not "Defer". Deferring cheap work just creates tracking overhead and things that get forgotten.

Tag each item with one of these labels so the user can triage quickly.

### 7. Mistake Pattern Check

Scan the conversation for repeated mistakes, backtracking, corrections, or "no, that's wrong" moments. If any pattern emerges — the same type of error happening more than once, or a mistake that suggests a systemic gap — flag it as a candidate for a never-again rule:

```
### Never-Again Candidates
- **{pattern}** — {brief description of what kept going wrong and why it might be systemic}
```

If no mistake patterns are found, omit this section entirely. Don't force it — clean sessions are fine.

### 8. Session Memory Note

If session memory files exist (`/memories/session/*`), mention them — the user may want to preserve specific notes before the session ends. Don't create files on the user's behalf; just flag what's there.

## Constraints

- **Read-only on the codebase.** This skill reads files to verify escalations but never modifies code, plans, or docs.
- **Don't be exhaustive about obvious completions.** If a task was clearly finished and verified, don't list it. Focus on the gaps.
- **Don't fabricate escalations.** If the session was clean, say so. A 🟢 verdict is perfectly fine.
- **Don't create memory files.** The sweep reports in chat only. The user decides what (if anything) to persist — plans, memory notes, or nothing.
- **Git state is out of scope.** Never check `git status`, uncommitted changes, or commit history. Never flag uncommitted work, missing branches, or "you should commit this" as escalations. The user manages branches across multiple parallel sessions — what's committed or not is their concern, not the sweep's. The sweep reports on *conversation* escalations (unfinished features, open design questions, missing tests), not on working-tree hygiene.
