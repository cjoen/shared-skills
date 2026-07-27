---
name: codebase-research
description: Deep dive into a codebase area or implementation and produce a research document in ai-context/research/
argument-hint: <topic> (e.g., "auth flow", "CI pipeline", "Redux generators", "navigation")
---

# Codebase Research

Conduct a deep-dive investigation into a specific codebase area and produce a research document capturing findings.

## Input

If `$ARGUMENTS` is empty, ask what area to research.

If `$ARGUMENTS` is provided, treat it as the topic. Incorporate any additional context or specific questions from the user message.

## Phase 1: Orient

Quick orientation before exploration:

1. **Check ai-context/** — if it exists, read `ai-context/README.md` and any docs in `ai-context/research/` (`ls ai-context/research/ 2>/dev/null` — no output means the folder doesn't exist yet and this is the first research run) to know what's already documented. Avoid duplicates. If `ai-context/` doesn't exist at all, note at the end that running `/update-context init` would give this research a fuller home. This skill **owns** the `research/` folder: `/update-context init` never creates it, so `/codebase-research` creates it on its first run (see Phase 4).
2. **Check for project context files** — scan for CLAUDE.md, MEMORY.md, or similar files with project-specific notes relevant to the topic.
3. **Determine scope** — identify 2-3 distinct investigation angles for parallel exploration.

If existing research already covers the topic, tell the user and ask: update or fresh investigation?

## Phase 2: Explore

Launch **up to 3 Explore agents in parallel** (single message, multiple tool calls). Each agent gets a distinct, non-overlapping focus.

**Agent prompt guidelines:**
- Be specific about each agent's target — don't give all agents the same broad prompt
- Tell agents to read files fully, not just find them
- Ask agents to note file paths, configurations, concrete details
- Direct agents to specific directories or file types when possible

**Typical investigation split patterns:**

| Topic Type | Agent 1 | Agent 2 | Agent 3 |
|---|---|---|---|
| **system/pipeline** (CI, deployment, dependency management) | Configuration files and tool setup | Workflow/pipeline definitions and triggers | Integration points, notifications, post-action hooks |
| **feature/domain** (auth, navigation, payments) | Core implementation and data flow | Integration points, providers, dependencies | Tests, edge cases, error handling |
| **pattern/architecture** (Redux generators, API clients) | Implementation and type definitions | Usage patterns across the codebase | Configuration, conventions, constraints |

Use fewer agents when the topic is narrow (1-2 files) or well-scoped.

## Phase 3: Verify

After agents return, **directly read 3-5 key files** yourself to:
- Confirm critical details from agent findings
- Fill gaps where agents were vague or conflicting
- Capture exact configurations, code patterns, values for the document

Do not skip this — agent findings are summaries that may miss nuance.

## Phase 4: Write

**First, make sure `ai-context/` is git-ignored.** This skill can create `ai-context/`
before `/update-context init` has ever run, so it must apply the same guard rather than
assuming init did it:

```bash
REPO_ROOT=$(git rev-parse --show-toplevel)
cd "$REPO_ROOT"
if git check-ignore -q ai-context/ 2>/dev/null; then
  echo "already ignored"
elif [ -f .gitignore ]; then
  printf '\n# AI context docs (local only)\nai-context/\n' >> .gitignore
else
  printf '\n# AI context docs (local only)\nai-context/\n' >> .git/info/exclude
fi
git check-ignore -q ai-context/ && echo IGNORED || echo NOT_IGNORED
```

If that prints `NOT_IGNORED`, stop and tell the user instead of writing into the repo.

Then ensure the `ai-context/research/` folder exists — create it on this first run if it doesn't (`mkdir -p ai-context/research`). Then create the research document at `ai-context/research/<topic>-research.md`.

**Naming:** lowercase, hyphenated, suffixed with `-research.md`. Examples:
- `auth-flow-research.md`, `ci-pipeline-research.md`, `payment-service-research.md`

### Document structure

Follow the established format from existing research docs:

```markdown
# <Topic Title>

<One-sentence description of what this document covers and its scope.>

## 1. <First Major Section>

<Content organized with tables, lists, code blocks. Be specific — real file paths, real config values, real function names.>

## 2. <Second Major Section>

...

## N. Key Source Files

| File | Role |
|---|---|
| `path/to/file` | One-line description |
```

### Writing rules

- **Be specific** — real paths, real values, real function signatures. Every path must exist.
- **Tables over prose** — for inventories, config values, file listings, comparison matrices.
- **Code blocks for configs** — show actual snippets, not descriptions.
- **Section per concern** — each numbered section covers one logical area. Cross-reference, don't repeat.
- **End with a file reference table** — every key source file mentioned.
- **Include a gaps/issues section** if you found weak points, missing coverage, undocumented behavior. Often the most valuable part.
- **No footer date** — research docs are point-in-time snapshots, not maintained docs like ai-context core docs.

## Phase 5: Confirm

After writing, briefly summarize for the user:
- What the document covers (2-3 sentences)
- Key findings or surprises worth highlighting
- Gaps or open questions unresolvable from code alone

## Anti-patterns

- **Don't duplicate ai-context core docs** — if `architecture.md` already covers something, reference it instead of restating it in the research doc.
- **Don't write a document that's just a file listing** — research docs should explain *how things work and why*, not just *what exists*.
- **Don't leave agent findings unverified** — always read key files yourself before writing.
- **Don't over-scope** — a research doc should be focused enough to read in 5-10 minutes. If the topic is too broad, suggest splitting into multiple docs.
