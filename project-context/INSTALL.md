# Project Context Skills

Two Claude Code skills for building and maintaining AI context docs — efficient codebase roadmaps that reduce token waste and give agents better guidance.

> **Quick start:** paste this entire doc into a Claude Code session and say "install both skills." Claude will fetch the skill files from GitHub and write them to your skills folder.

## Why

High-quality context docs mean agents spend fewer tokens rediscovering things you already know. The pattern comes from the Research → Plan → Implement workflow in [this article by Boris Tane](https://boristane.com/blog/how-i-use-claude-code/) — these skills make Step 1 easy.

| Skill | Purpose |
|---|---|
| `/update-context` | Creates and maintains high-level architecture docs agents use for navigation. Enough context for smaller tasks on its own. |
| `/codebase-research` | Deep-dives a specific component before a complex task. Maps key functions, conventions, and structure so agents know exactly where to edit and how new code should align. |

## Typical workflow

Example: adding a feature to the account → user detail page.

1. Pull new code → run `/update-context` to refresh context (fast after initial setup)
2. Run `/codebase-research account detail page` → Explore agents investigate, output `account-detail-research.md`
3. Start a new session, reference the research doc by path in your first message, begin planning
4. Iterate on the plan until it's detailed, then implement

Step 3 is a deliberate checkpoint — referencing the research doc cuts the redirect cycles that eat tokens when an agent has to discover the relevant area from scratch. You can also request a human-readable version of the research doc to get up to speed on unfamiliar codebases.

## Install

### Option A — Claude installs for you

Paste this entire document into a Claude Code session and say: "Install both skills." Claude will fetch and write the following files:

- `~/.claude/skills/update-context/SKILL.md` from `https://raw.githubusercontent.com/cjoen/shared-skills/main/project-context/update-context/SKILL.md`
- `~/.claude/skills/codebase-research/SKILL.md` from `https://raw.githubusercontent.com/cjoen/shared-skills/main/project-context/codebase-research/SKILL.md`

### Option B — Manual

```bash
cp -r update-context ~/.claude/skills/
cp -r codebase-research ~/.claude/skills/
```

## Setup

### 1. Add project context loading to your global Claude instructions

In `~/.claude/CLAUDE.md`, add:

```markdown
## Check for Project Context First

Before exploring code, making assumptions, or launching Explore agents, find the
ai-context directory. Priority order:

1. Check this file's "AI Context Paths" section for an entry matching the current repo root
2. `ai-context/` directory at repo root — read `README.md` inside, then load relevant docs
3. `ai-context.md` — flat-file variant for smaller projects
4. `CONTEXT.md` — common single-file convention

If none exist and the task involves code exploration or research, suggest the
`/update-context` skill to bootstrap one.
```

### 2. Initialize project context

Run the init — use Opus for the first run, Sonnet handles updates fine:

```
/update-context init
```

After exploring the codebase, the skill will ask where to place `ai-context/`:

- **Sibling folder** (recommended if team isn't onboarded yet) — e.g. `~/work/ai-context/<project-name>/`. Stays out of the repo, no `.gitignore` needed.
- **Inside the repo** — `<repo-root>/ai-context/`. Add `ai-context/` to `.gitignore` until the team is onboarded.

You can also skip the prompt by passing the path inline: `/update-context init ~/work/ai-context/myproject`.

## Commands

### `/update-context`

| Command | When |
|---|---|
| `/update-context init` | Once at project start — deep exploration, writes all initial context docs |
| `/update-context` | After major pulls, ~1–2× per week — refreshes stale docs against git history |
| `/update-context correct <description>` | When a doc has a specific error — targeted fix, no full sweep |

Correction example: `/update-context correct conventions.md says snake_case but the project uses kebab-case`

### `/codebase-research <topic>`

Deep-dives a specific area and writes `ai-context/research/<topic>-research.md`. Use this before any complex task in a brownfield area. Reference the output doc by path when starting your planning session.

Works best when `ai-context/` is already initialized, but can run without it — it will skip the orientation step and note that `/update-context init` would improve future runs.

