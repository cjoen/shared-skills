---
name: update-context
description: Create or update ai-context docs to keep AI agents aligned with the current codebase
argument-hint: <doc-or-topic> | "init" | "correct <description>" (e.g., "navigation", "init", "correct api.md says JWT but we use OAuth")
---

# AI Context Manager

Maintain an `ai-context/` directory giving AI agents deeper project understanding than CLAUDE.md provides.

Run `init` once at project start. Run `update` after sprints or dependency bumps. Use `correct` when you catch a specific error in a doc.

## Input

If `$ARGUMENTS` is `init`, run the **Init** workflow.

If `$ARGUMENTS` starts with `correct`, run the **Correct** workflow with the remainder as the correction description.

If `$ARGUMENTS` names a doc or topic (e.g., `api`, `conventions`), focus the **Update** workflow on it.

If `$ARGUMENTS` is empty, run the **Update** workflow across all docs.

## Environment Detection

Resolve the working environment first:

1. **Repo root**: Run `git rev-parse --show-toplevel 2>/dev/null`. Store as `$REPO_ROOT`.
2. If not in a git repo, ask: "I can't detect a git repository. What is the project root directory?"
3. **ai-context path**: Resolve as `$REPO_ROOT/ai-context/` (or user-provided root + `/ai-context/`).
4. All git commands and relative paths use `$REPO_ROOT`.

# Init Workflow

Create `ai-context/` at the project root with context docs from thorough codebase exploration.

## Phase 1: Deep Exploration

Explore the project. Adapt investigation to project type (backend, frontend, CLI, library, monorepo, pipeline, infra, etc.).

| Area | What to Investigate |
|------|---------------------|
| Project identity | What is this, who uses it; language(s), framework(s), runtime(s), deployment targets |
| Architecture | Top-level structure; org pattern (monorepo, layered, microservices); module/package catalog; import resolution; key dependencies |
| Data layer | Databases, ORMs, migrations, state management, message passing, data flow, provider composition |
| External interfaces | APIs consumed and exposed; auth patterns; env config; feature flags |
| User-facing structure | Routes, screens, commands, or public API surface depending on project type |
| Conventions | Naming, linting, formatting, testing patterns, commit/branching, module creation, CI/CD structure |
| Testing infrastructure | Frameworks, test data patterns, how to run tests |

Read key files, grep patterns, check configs. Don't guess — accuracy is the point.

## Phase 2: Ignore Before Writing

**This step is mandatory and must run before you create any file.** `ai-context/` lives
at the repo root but is for AI agents on this machine, not the shared repo. Ignoring it
first means it is never briefly visible as untracked changes in the user's `git status`.

Run this exactly:

```bash
cd "$REPO_ROOT"
if git check-ignore -q ai-context/ 2>/dev/null; then
  echo "already ignored"
elif [ -f .gitignore ]; then
  printf '\n# AI context docs (local only)\nai-context/\n' >> .gitignore
  echo "added to .gitignore"
else
  printf '\n# AI context docs (local only)\nai-context/\n' >> .git/info/exclude
  echo "added to .git/info/exclude"
fi
```

Prefer `.gitignore` when one exists. Fall back to `.git/info/exclude` only when the repo
has no `.gitignore` — `info/exclude` is local-only and never committed, so it is the
safe choice on a repo whose team has not adopted this workflow.

**Verify before continuing:**

```bash
git -C "$REPO_ROOT" check-ignore -q ai-context/ && echo IGNORED || echo NOT_IGNORED
```

If this prints `NOT_IGNORED`, stop and tell the user rather than writing docs into a
repo that will pick them up as untracked files. Note that editing `.gitignore` leaves a
modified tracked file in the user's working tree — mention this in your summary so they
can decide whether to commit that one-line change.

## Phase 3: Create Documents

Create `ai-context/` at the repo root with docs tailored to what this project has. A small CLI might need only `context/architecture.md` + `context/conventions.md`; a large monorepo may need all. Use judgment.

**Layout:** `README.md` stays at the `ai-context/` root — it is the index. Every core context doc goes under an `ai-context/context/` subfolder. Do **not** create a `research/` folder during init: that folder is owned by the `/codebase-research` skill, which creates it on its first run.

```
ai-context/
  README.md            # index + Document Scopes table — stays at root
  context/             # all core context docs live here
    architecture.md
    conventions.md
    ...                # data-layer, api, navigation, testing, etc. as applicable
```

### Always create:

**`ai-context/README.md`** — Index of all context files. Include:
- Project identity (name, stack, what it does, one paragraph)
- Table of all context docs with one-line descriptions
- **Scope table**: maps each doc to codebase paths it watches (Update workflow uses this for staleness)
- Note any existing human docs
- Footer: `*Last updated: YYYY-MM-DD | Verified against: <version-tag-or-short-commit>*`

Example scope table:
```markdown
## Document Scopes

| Document | Codebase paths watched |
|----------|----------------------|
| `context/architecture.md` | top-level dirs, dependency manifests, module configs |
| `context/data-layer.md` | `src/models/`, `src/db/`, `migrations/` |
| `context/api.md` | `src/routes/`, `src/middleware/`, `src/auth/` |
```

**`ai-context/context/architecture.md`** — Most important doc:
- Tech stack table (layer → technology + version)
- Top-level directory map with annotations
- Module/package/service taxonomy (categorized, one-line descriptions)
- Import/module resolution conventions
- Build and deploy tooling overview

### Create if applicable (all under `ai-context/context/`):

- **Data layer** — Database schemas, ORMs, state stores, caching, data flow patterns
- **External interfaces / API layer** — Clients consumed, APIs exposed, auth, environment config
- **User-facing structure** — Routes, screens, commands, public API surface
- **Conventions** — Naming, linting, testing, commits, module creation
- **Testing** — Test infrastructure, patterns, tooling (if complex enough to warrant its own doc)
- **Feature flags** — Flag system, current flags, defaults
- **Infrastructure** — Deployment, CI/CD, containerization, cloud resources (if substantial)

### Document format rules:

- **Real paths only** — every file/dir path must exist in the codebase
- **Cross-reference, don't duplicate** — one doc per domain; link to others instead of restating

Footer on every doc: `*Last updated: YYYY-MM-DD | Verified against: <latest-tag-or-short-commit>*`

## Phase 4: Validate

Apply the validation rules from Update workflow Phase 4.

## Output Format

### Documents Created
[List of documents with one-line description of each]

### Key Architectural Insights
[Notable findings from exploration]

### Areas to Clarify
[Anything unclear that the developer should verify]

# Update Workflow

Compare `ai-context/` docs against the current codebase; update stale ones.

## Phase 1: Compare Freshness

If `ai-context/` doesn't exist: tell the user to run `/update-context init` first.

For each core doc (`README.md` at the root plus every doc under `ai-context/context/`):

1. Extract `Last updated` date and version/commit from footer. **Missing or unparseable footer = stale.**
2. Determine current repo state:
   ```bash
   git tag --sort=-version:refname | head -5
   git log -1 --format='%H %ai %s'
   git rev-list --count --since='YYYY-MM-DD' HEAD
   ```
3. Use the **scope table** in `ai-context/README.md` to find each doc's watched paths
4. Doc is **stale** if any commits since last update touch its scope paths; **current** otherwise.

## Phase 2: Find the Diff

For each stale document, find what changed within its scope:

```bash
# By tag (preferred)
git log <old-tag>..HEAD --oneline --stat -- <path1> <path2> ...

# By date (fallback)
git log --since='YYYY-MM-DD' --oneline --stat -- <path1> <path2> ...

# New files added / removed
git diff --name-only --diff-filter=A <old-tag>..HEAD -- <scope-paths>
git diff --name-only --diff-filter=D <old-tag>..HEAD -- <scope-paths>
```

**Efficiency tip**: Start narrow — 5-6 high-signal paths (new/removed modules, configs, entry points). Expand only if initial scan suggests more.

### Classify changes

**Meaningful** (require doc update):
- New/removed/renamed modules, packages, services, or major components
- Changes to public APIs, data models, schemas, or type signatures
- New/changed routes, endpoints, commands, or navigation structure
- New/removed/renamed feature flags, environment variables, or config keys
- Changes to middleware/provider/interceptor composition order
- New/changed auth flows or config structure
- New/changed lint rules, generators, conventions, or commit hooks
- Changes to test infrastructure (frameworks, patterns, helpers)
- Major dependency version bumps that change developer-facing APIs
- Infrastructure changes that affect how the project is built or deployed

**Noise** (skip):
- Bug fixes that don't change architecture or APIs
- Patch/minor dependency bumps with no API changes
- Cosmetic refactors (renames within a file, formatting)
- New unit tests or test data
- CI/CD changes (unless they change documented developer commands)

## Phase 3: Update Stale Docs

For each document with meaningful changes:

1. **Preserve structure** — heading hierarchy, tables, section order
2. **Preserve tone** — imperative, concise, no filler
3. **Preserve scope** — don't add info that belongs elsewhere
4. **Add, don't bloat** — match existing item format
5. **Remove stale references** — delete entries for things gone
6. **Update code examples** — verify imports and API signatures are current
7. **Update footer** — today's date, latest tag/commit

Verify cross-doc consistency after:
- `README.md` scope table current (paths valid, new docs listed)
- Cross-references point to existing docs
- Shared path refs consistent across docs

## Phase 4: Validate

For every updated document:
- All file/dir paths exist in the codebase
- No duplicate info across docs
- No contradictions with other docs or codebase
- Tables (modules/flags/routes/models/etc.) complete; removed items excluded
- Code examples use correct syntax + current API signatures
- Footer updated

## Output Format

### Documents Checked
[List every context doc: current or updated]

### Changes Made
[For each updated doc: brief list of additions, removals, changes]

### Validation
[Confirm all checks passed, or list unresolved issues]

# Correct Workflow

Fix a specific incorrect assumption in ai-context docs without a full git-diff sweep.

## Input

`$ARGUMENTS` starts with `correct`. The remainder is the correction — what was wrong and what's right. Example: `correct architecture.md says we use Redux Saga but we use Redux Toolkit`

If no description follows `correct`, ask: "What assumption needs correcting?"

## Phase 1: Locate

Search ai-context docs for the incorrect claim:

1. Extract key terms from the correction (the wrong thing, not the right thing)
2. Grep all ai-context docs for those terms:
   ```bash
   grep -ri "<key-terms>" $REPO_ROOT/ai-context/
   ```
3. Read each matching section in full — don't rely on grep snippets
4. List every doc and section containing the assumption

If no matches: tell the user where you searched and ask them to point to the doc.

## Phase 2: Verify

Before editing, confirm the correct state from code:

1. Identify codebase paths most likely to confirm the truth (configs, entry points, key files)
2. Read those files directly — the user's phrasing may not match exact code values
3. Note precise values, names, or patterns the docs should reflect

**If the code confirms the doc is already correct:** stop. Tell the user what you found and which files you checked. Do not edit. Ask if they want to proceed anyway or if the correction description needs refinement.

Skip verification only if the correction is organizational (wrong doc, wrong section) rather than factual.

## Phase 3: Edit

For each affected section:

1. Minimum edit — change only the incorrect claim and its immediate context
2. Do not restructure surrounding content or fix unrelated issues
3. If the correction reveals an undocumented truth, add it
4. Update the footer date on every doc touched

## Phase 4: Cross-Doc Check

After editing, check for the same assumption elsewhere:

```bash
grep -ri "<wrong-term>" $REPO_ROOT/ai-context/
```

If found in other docs, apply the same correction there.

## Output Format

### Assumption Corrected
[One line: what was wrong → what's right]

### Docs Updated
[Each doc: section changed and what changed]

### Verified Against
[Files read to confirm the correct state]

### Also Fixed
[Cross-doc instances corrected, or "none"]
