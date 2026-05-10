---
name: changelog
description: Generate a structured, human-readable CHANGELOG from git commit history, strictly following the Keep a Changelog 1.1.0 format (keepachangelog.com). Use this skill whenever the user wants to generate a changelog, release notes, or commit summary — even if they phrase it as "what changed", "summarise recent commits", "write release notes for v2.1", "CHANGELOG for this sprint", or "what's new since last tag". Trigger any time git commits or version history are involved in producing written output.
---

# Changelog Generator Skill

Generate changelogs that strictly follow the [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) format.

This skill works from **commit subjects only** — no diffs, no `--stat`, no file inspection. This keeps it fast and honest: the quality of the output reflects the quality of the commit messages.

## Step 0 — Ask for audience context before doing anything

Before running any git commands, ask the user one question:

> "Who's the primary audience for this changelog — end users, developers/API consumers, or internal team/ops?"

This changes the language significantly:
- **End users**: plain English, no technical terms, focus on visible behaviour ("You can now export reports as PDF")
- **Developers**: can reference APIs, modules, config changes ("Added `exportReport(format)` to the SDK")
- **Internal/ops**: can mention infra, deploy steps, env var changes ("Requires `REDIS_URL` env var on deploy")

If the user is in a hurry and says skip it, default to **developers**.

## Keep a Changelog 1.1.0 — Rules (strictly follow these)

### File structure

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.0] - 2023-03-05
### Added
- ...

## [1.0.0] - 2023-01-01
### Fixed
- ...

[Unreleased]: https://github.com/org/repo/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/org/repo/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/org/repo/releases/tag/v1.0.0
```

### The six section types — use ONLY these, in this order

| Section | Maps from Conventional Commits |
|---------|-------------------------------|
| `Added` | `feat:`, `feature:` |
| `Changed` | `refactor:`, `perf:`, `style:`, behavioural changes |
| `Deprecated` | `deprecate:` or commit mentioning deprecation |
| `Removed` | `remove:`, `drop:` |
| `Fixed` | `fix:`, `bugfix:`, `hotfix:`, `revert:` |
| `Security` | `security:`, CVE fixes |

**Maintenance commits** (`chore:`, `build:`, `ci:`, `test:`, `docs:`) — include only if clearly notable (e.g. "Upgrade PostgreSQL from 14 to 16"); skip trivial ones silently.

### Formatting rules

- **Omit empty sections entirely**
- **No emojis** in section headers or bullet points
- **No SHAs** in bullet points
- **No PR numbers** unless the user asks
- Version in square brackets: `[1.2.0]` not `v1.2.0`
- Date in ISO 8601: `2026-05-10`
- `[Unreleased]` section always present at the top (empty if nothing pending)
- Newest release first (reverse chronological)
- **Breaking changes**: inline in the relevant section with a `**Breaking:**` prefix
- Compare links block at the very bottom

## Step-by-step workflow

### 1. Discover the repo and range

```bash
cd <repo_path>

# List tags to help determine range
git tag --sort=-creatordate | head -10

# Default range: last tag to HEAD
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
RANGE="${LAST_TAG}..HEAD"
[ -z "$LAST_TAG" ] && RANGE="HEAD~30..HEAD"  # fallback: no tags

# Get remote URL for compare links
git remote get-url origin
```

### 2. Fetch commit subjects only (no diffs)

One command, subjects and trailers only:

```bash
git log $RANGE \
  --pretty=format:"%s|%b|%ad" \
  --date=short \
  --no-merges
```

Fields: `subject | body | date`

The body is fetched only to detect `BREAKING CHANGE:` trailers and `!` breaking markers — not for content.

### 3. Filter: skip noise commits

Skip a commit entirely if the subject matches any of these patterns:

| Pattern | Examples |
|---------|---------|
| Merge commits | `Merge branch`, `Merge pull request` |
| WIP / incomplete | `wip`, `wip:`, `squash!`, `fixup!` |
| Version bumps | `bump version`, `release v`, `chore: v1.2.3`, subject is just a semver |
| CI skip | `[skip ci]`, `[ci skip]` |
| Trivial chore | `fix typo`, `address review comments`, `update snapshot`, `lint`, subject under 10 chars |
| Empty/junk | `oops`, `test`, `asdf`, `tmp` |

**Do not silently discard.** Keep a list of every skipped commit. After generating the changelog, report them:

> **Skipped commits (12)** — review these to make sure nothing important was missed:
> - `fix typo in README` (trivial)
> - `wip: auth refactor` (incomplete)
> - `Merge pull request #42` (merge)
> - ...

### 4. Group related commits into single entries

Before writing entries, look for commits that belong to the same logical change and collapse them into one:

- Same scope + same section: `feat(export): add export button`, `feat(export): add export progress`, `fix(export): wrong filename` → single "Add report export" entry under `Added`, with the fix folded in
- Revert pairs: a commit and its revert cancel each other out → skip both, mention in the skipped list
- Series of small `fix:` commits on the same thing → one entry describing the end state

Use your judgement. When in doubt, keep them separate.

### 5. Rewrite subjects into human-readable entries

**Never copy the raw commit subject verbatim.** Rewrite each entry for the target audience:

| Raw commit | End users | Developers | Internal/ops |
|-----------|-----------|------------|--------------|
| `feat: add exportToPDF()` | "Export any report as a PDF file" | "Add `exportToPDF(format?)` method to ReportService" | "New PDF export endpoint: `POST /reports/:id/export`" |
| `fix: null pointer on login` | "Fix crash when logging in without a verified email" | "Fix null pointer in `AuthService.login()` when user has no email record" | "Fix login crash affecting ~3% of users since v2.1.0" |
| `perf: cache permission lookups` | _(skip — not user-visible)_ | "Cache user permission lookups — reduces API response time ~40%" | "Redis-backed permission cache; requires `REDIS_URL` on deploy" |

Rules:
- Strip conventional commit prefix (`feat:`, `fix:`, etc.)
- Start with a capital verb in imperative mood: "Add", "Fix", "Remove", "Update"
- Lead with impact, not implementation
- Keep entries to one line (~80 chars max)
- If a commit is ambiguous (no prefix), classify by reading the subject text, defaulting to `Changed`

### 6. Write the changelog entry

```markdown
## [2.3.0] - 2026-05-10

### Added
- Export any report as a PDF file
- Dark mode support across all dashboard views

### Changed
- **Breaking:** Session tokens now expire after 1 hour (previously 24 hours)
- Permission checks now use a Redis cache, significantly reducing API latency

### Fixed
- Fix crash when logging in without a verified email
- Fix incorrect timestamps in scheduled job history
```

Compare link:
```
[2.3.0]: https://github.com/org/repo/compare/v2.2.0...v2.3.0
```

If the remote URL is unknown, omit compare links and note that the user should add them.

### 7. Output

- **Default**: print the release block in chat for review, followed by the skipped commits list
- **"Save it"**: prepend to `CHANGELOG.md` (after the header, before previous releases). Confirm first: "Ready to prepend this to CHANGELOG.md?"
- **"Full file"**: generate complete `CHANGELOG.md` with standard header and all tag history

## Edge cases

- **Monorepos**: ask whether to filter by path (`git log -- packages/api/`) or per-package
- **No tags**: fall back to last 30 commits, label version `[Unreleased]`
- **Squash merges**: PR title is the subject — use it as-is, ignore the squashed commit list in the body
- **Pre-release**: `[2.0.0-beta.1] - 2026-05-10`
- **Yanked release**: `## [1.2.0] - 2026-04-01 [YANKED]`