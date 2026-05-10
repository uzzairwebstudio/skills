---
name: conventional-commits
description: Write, validate, and fix git commit messages following the Conventional Commits 1.0.0 specification. Use this skill whenever the user wants to commit changes, write a commit message, fix a bad commit message, or asks "what should my commit message be?". Also triggers when the user says "commit this", "stage and commit", "commit my changes", or pastes a commit message that looks malformed. Pairs with the changelog skill — good commits produce good changelogs.
---

# Conventional Commits Skill

Help write, validate, and fix commit messages following the [Conventional Commits 1.0.0](https://conventionalcommits.org) specification.

## Commit message format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Use for | SemVer impact |
|------|---------|---------------|
| `feat` | New feature | MINOR bump |
| `fix` | Bug fix | PATCH bump |
| `perf` | Performance improvement | PATCH bump |
| `refactor` | Code change, no new feature or fix | none |
| `docs` | Documentation only | none |
| `test` | Adding or fixing tests | none |
| `build` | Build system, dependencies | none |
| `ci` | CI/CD configuration | none |
| `chore` | Maintenance, tooling, config | none |
| `style` | Formatting, whitespace (no logic change) | none |
| `revert` | Reverts a previous commit | none |

### Breaking changes

Two valid ways to mark a breaking change:

```
feat!: remove deprecated login endpoint
```

```
feat(auth): redesign session handling

BREAKING CHANGE: session tokens now expire after 1 hour instead of 24 hours
```

Both correlate with a **MAJOR** SemVer bump.

### Scope

Optional. Identifies the part of the codebase affected. Use lowercase, short, consistent names across the team:

```
fix(auth): handle null user on login
feat(payroll): add EPF contribution calculator
chore(deps): upgrade Laravel to 11.x
```

### Description rules

- Lowercase, imperative mood: "add feature" not "Added feature" or "Adds feature"
- No period at the end
- Max ~72 characters
- Must complete the sentence: "This commit will ___"

### Body (optional)

- Separated from description by a blank line
- Explain *why*, not *what* (the diff shows what)
- Wrap at 72 characters

### Footer (optional)

- Separated from body by a blank line
- Format: `token: value` or `token #value`
- Common tokens: `BREAKING CHANGE`, `Closes`, `Refs`, `Co-authored-by`

```
fix(jobs): prevent duplicate application submissions

Added a unique constraint check before inserting to prevent race
conditions when users double-click the apply button.

Closes #142
```

---

## Workflow

### Mode 1 — Generate a commit message from staged changes

When the user says "commit this" or "write a commit message":

1. Run `git diff --staged --stat` to see what files changed
2. Run `git diff --staged` to read the actual changes (subjects only — don't over-index on implementation details)
3. Determine the most appropriate type and scope from the changes
4. Write the commit message
5. Show the message and ask for confirmation before running `git commit`

```bash
# Check what's staged
git diff --staged --stat

# Read the changes
git diff --staged
```

Then propose:

```
feat(auth): add password reset via email OTP

Users can now reset their password without contacting support.
A 6-digit OTP is sent to the registered email, valid for 15 minutes.

Closes #88
```

Ask: "Ready to commit with this message?"

If confirmed:
```bash
git commit -m "feat(auth): add password reset via email OTP" \
  -m "Users can now reset their password without contacting support." \
  -m "A 6-digit OTP is sent to the registered email, valid for 15 minutes." \
  -m "Closes #88"
```

### Mode 2 — Validate an existing commit message

When the user pastes a commit message or asks "is this a good commit?":

Check against these rules and report clearly:

| Check | Rule |
|-------|------|
| Type | Must be one of the valid types |
| Format | Must match `type(scope): description` |
| Case | Description must be lowercase |
| Tense | Must be imperative mood |
| Length | Description ≤ 72 chars |
| Period | No trailing period |
| Breaking | `BREAKING CHANGE` footer or `!` if breaking |

Show a clear pass/fail for each, then provide a corrected version if anything fails.

**Example:**

Input: `"Fixed the login bug."`

Output:
```
❌ Missing type prefix
❌ Past tense ("Fixed" → "fix")
❌ Trailing period
❌ No scope

Suggested fix:
  fix(auth): resolve null pointer crash on login
```

### Mode 3 — Fix a recent commit message

When the user says "fix my last commit message" or "amend this":

```bash
# Show the last commit
git log -1 --pretty=format:"%s%n%n%b"
```

Analyse, propose a corrected message, then:

```bash
git commit --amend -m "corrected message"
```

Warn if the commit has already been pushed: amending rewrites history, which requires a force push.

### Mode 4 — Suggest a scope convention for the repo

When the user asks "what scope should I use?" or "what scopes do we have?":

```bash
# Extract scopes used in the last 50 commits
git log -50 --pretty=format:"%s" | grep -oP '(?<=\().*?(?=\))' | sort | uniq -c | sort -rn
```

List the existing scopes and suggest the right one based on context.

---

## Good vs bad examples

| Bad | Good |
|-----|------|
| `fix bug` | `fix(auth): handle null session on logout` |
| `updated styles` | `style(dashboard): reformat sidebar layout` |
| `WIP` | *(stage partial work, commit with `chore: wip [description]` only if necessary)* |
| `Fixed the thing.` | `fix(jobs): prevent duplicate application on double-click` |
| `feat: Added new login page` | `feat(auth): add email OTP login page` |
| `misc changes` | Split into separate commits per concern |

---

## Multi-commit advice

If staged changes cover multiple concerns (e.g. a bug fix AND a new feature), suggest splitting:

```bash
# Stage only part of the changes
git add -p

# Commit the first concern
git commit -m "fix(auth): ..."

# Stage the rest
git add -p

# Commit the second concern
git commit -m "feat(dashboard): ..."
```

One commit = one logical change. If you can't write a single-type message, it probably needs splitting.

---

## SemVer cheatsheet

| Commit type | Version bump |
|------------|-------------|
| `fix`, `perf` | 1.2.0 → 1.2.1 |
| `feat` | 1.2.0 → 1.3.0 |
| `feat!` or `BREAKING CHANGE` | 1.2.0 → 2.0.0 |
| `chore`, `docs`, `style`, `test`, `refactor`, `ci`, `build` | no bump |