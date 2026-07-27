# project-context

Two Claude Code skills for building and maintaining AI context docs — codebase roadmaps
that cut token waste and give agents better guidance before they start exploring or
planning.

| Skill | Purpose |
|---|---|
| `/update-context` | Creates and maintains `ai-context/` — high-level architecture docs agents use for navigation. Often enough on its own for smaller tasks. |
| `/codebase-research` | Deep-dives a specific area before a complex task. Maps key functions, conventions, and structure so agents know exactly where to edit and how new code should align. |

**To install, see [INSTALL.md](INSTALL.md)** — it doubles as a paste-into-Claude install
script.

## Why

In a brownfield project, most of an agent's first few thousand tokens go to
rediscovering things you already know. `/update-context` gives it a map up front.
That's usually enough for small tasks — but for anything complex you want a clear
picture of the specific component you're touching, which is what `/codebase-research`
produces: where edits need to be made, and what conventions new code should match.

The pattern comes from the Research → Plan → Implement workflow in
[this article by Boris Tane](https://boristane.com/blog/how-i-use-claude-code/), which
inspired these skills and is worth reading first.

1. **Research** — `/codebase-research`
2. **Plan** — conversational level-setting on the desired outcome
3. **Review** — review the agent's plan and iterate until it's genuinely detailed
4. **Implement** — the agent follows the plan

These skills make Step 1 easy.

## Typical workflow

Example: adding a component to the account → user detail page.

1. Pull new code → run `/update-context` to refresh (fast once initialized)
2. Run `/codebase-research account detail page` → Explore agents investigate, output lands at `ai-context/research/account-detail-page-research.md`
3. **Start a new session.** Reference the research doc by its exact path in your first message, then begin planning
4. Iterate on the plan until it's detailed, then implement

Step 3 is a deliberate checkpoint. Referencing the doc by path cuts the redirect cycles
that eat tokens when an agent has to discover the relevant area from scratch. You can
also ask for a human-readable version of the research doc — it's a fast way to get up to
speed on an unfamiliar codebase or component yourself.

## What it creates

Everything lives at the repo root. Both skills git-ignore `ai-context/` before writing
anything — via `.gitignore`, or `.git/info/exclude` when the repo has none — so you can
run this on a shared repo before the rest of the team is onboarded:

```
ai-context/
  README.md            # index + scope table (which doc watches which paths)
  context/             # core docs — architecture.md, conventions.md, etc.
  research/            # /codebase-research output, created on its first run
```

The scope table is what makes updates cheap: it maps each doc to the codebase paths it
watches, so `/update-context` can check git history and only refresh docs whose paths
actually changed.

## Commands

### `/update-context`

| Command | When |
|---|---|
| `/update-context init` | Once at project start — deep exploration, writes all initial context docs |
| `/update-context` | After major pulls, ~1–2× per week — refreshes stale docs against git history |
| `/update-context correct <description>` | When a doc has a specific error — targeted fix, no full sweep |

Correction example:
`/update-context correct conventions.md says snake_case but the project uses kebab-case`

Use Opus for the initial `init`; Sonnet handles routine updates fine.

### `/codebase-research <topic>`

Deep-dives a specific area and writes `ai-context/research/<topic>-research.md`. Use it
before any complex task in a brownfield area.

Works best when `ai-context/` is already initialized, but runs without it too — it'll
skip the orientation step and note that `/update-context init` would give future runs a
fuller home.
