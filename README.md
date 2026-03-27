<h1 align="center">🛩️ Wingman</h1>

<p align="center">
  <strong>AI development skills for GitHub Copilot, Claude Code, and other skills-compatible agents.</strong>
</p>

<p align="center">
  Reusable methodologies that make your AI coding assistant smarter — packaged as <a href="https://agentskills.io">Agent Skills</a>.
</p>

---

## What is Wingman?

Wingman is a collection of **AI development skills** — structured, tested methodologies that teach your AI assistant how to perform complex tasks reliably. Instead of repeating the same instructions every session, install Wingman once and your agent learns the process permanently.

Skills are built on the open [Agent Skills](https://agentskills.io) standard, so they work across:

- **VS Code** (GitHub Copilot)
- **Claude Code**
- **Copilot CLI**
- **Cursor**, **Gemini CLI**, and other compatible agents

## Skills

| Skill | What it does |
|-------|-------------|
| **`/sweep`** | Scans your chat session for incomplete work, deferred decisions, open questions, discovered issues, and untracked ideas — then produces a structured triage report so nothing falls through the cracks when you close a session. |
| **`/never-again`** | Root-causes a mistake and implements systemic repairs to prevent recurrence. Surveys your project's customization layers (instructions, skills, agents, memory, docs, lint) for the gap that allowed the error, recommends specific fixes, and implements them on approval. |

## Install

### VS Code

1. Open the Command Palette (`Ctrl+Shift+P`)
2. Run **Chat: Install Plugin From Source**
3. Paste: `https://github.com/dornstein/wingman`

Skills appear as `/` slash commands in chat. Type `/sweep` to invoke.

### Claude Code

```bash
/plugin marketplace add dornstein/wingman
/plugin install wingman@wingman
```

### Manual (any agent)

Clone the repo and copy the `skills/` directory to your agent's skills location:

```bash
git clone https://github.com/dornstein/wingman.git

# VS Code / Copilot
cp -r wingman/skills/* ~/.copilot/skills/

# Claude Code
cp -r wingman/skills/* ~/.claude/skills/
```

## How Skills Work

Each skill is a folder with a `SKILL.md` file containing structured instructions. When you invoke a skill (e.g., `/sweep`), your agent loads the instructions and follows the defined process step by step.

Skills use **progressive disclosure** — metadata loads at startup (~100 tokens), full instructions load on demand (~2K tokens), and supporting resources load only when needed. You can install many skills without consuming context.

## Philosophy

- **Encode process, not checklists.** Skills teach the *why* and *when*, not just the *what*. An agent with good skills makes judgment calls, not just checkbox passes.
- **Verify before reporting.** Skills that surface findings (like `/sweep`) cross-check against the actual codebase before presenting results. No stale escalations.
- **Bias toward action.** When a skill recommends next steps, it defaults to "do it now" for anything under 10 minutes. Deferring cheap work just creates debt.

## Contributing

Add a skill:

1. Create `skills/your-skill/SKILL.md` with the required frontmatter (`name`, `description`)
2. Follow the [Agent Skills specification](https://agentskills.io/specification)
3. Open a PR

## License

[MIT](LICENSE)
