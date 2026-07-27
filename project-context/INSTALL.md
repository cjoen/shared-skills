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

Before exploring code, making assumptions, or launching Explore agents, check for
project context files at the repo root. Priority order:

1. `ai-context/` directory — read `README.md` inside, then load relevant docs from `context/`
2. `ai-context.md` — flat-file variant for smaller projects
3. `CONTEXT.md` — common single-file convention

If none exist and the task involves code exploration or research, suggest the
`/update-context` skill to bootstrap one.
```

### 2. Initialize project context

Run the init — use Opus for the first run, Sonnet handles updates fine:

```
/update-context init
```

No prompts to answer — the skill explores the codebase and writes to `<repo-root>/ai-context/`:

```
ai-context/
  README.md            # index + scope table (which doc watches which paths)
  context/             # core docs — architecture.md, conventions.md, etc.
  research/            # created later by /codebase-research, not by init
```

**It stays out of your commits.** Before writing a single file, the skill runs
`git check-ignore` and, if `ai-context/` isn't already ignored, appends it to the repo's
`.gitignore` — or to `.git/info/exclude` when the repo has no `.gitignore`. It then
re-checks and refuses to write anything if the path still isn't ignored. Both
`/update-context` and `/codebase-research` apply this guard, so whichever you run first
is covered.

`.git/info/exclude` is local-only and never committed, which makes this safe on a repo
shared with a team that hasn't adopted the workflow. On a repo that does have a
`.gitignore`, note that the one-line addition is a modification to a tracked file —
yours to commit or revert.

### 3. Verify (optional)

```bash
git check-ignore -v ai-context/
```

Prints the file and line number of the matching rule. No output means it isn't ignored.

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

