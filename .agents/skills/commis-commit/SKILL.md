---
name: commis-commit
description: >
  Commit message conventions for CommiFlow based on Conventional Commits 1.0.0.
  Trigger: When creating, reviewing, or amending git commits, or when writing commit messages.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

## When to Use

Use this skill when:
- Creating a new commit
- Reviewing or amending existing commits
- Writing commit messages
- Setting up git hooks or commitlint

---

## Commit Format

```
<type>: <description>

[optional body]

[optional footer(s)]
```

### Rules

- `type` — lowercase, REQUIRED, from the allowed list
- `!` — appended before `:` to signal BREAKING CHANGE, OPTIONAL
- `:` — REQUIRED, followed by a single space
- `description` — short summary, REQUIRED, imperative mood, lowercase, no period at end
- Body — free-form, separated from description by one blank line, OPTIONAL
- Footer — `token: value` or `token #value`, separated from body by one blank line, OPTIONAL

---

## Allowed Types

| Type | Description | SemVer |
| :--- | :--- | :--- |
| `feat` | New feature | MINOR |
| `fix` | Bug fix | PATCH |
| `docs` | Documentation only | — |
| `style` | Formatting, whitespace, semicolons (no logic change) | — |
| `refactor` | Code restructure (no feature, no fix) | — |
| `perf` | Performance improvement | PATCH |
| `test` | Adding or updating tests | — |
| `build` | Build system, dependencies | — |
| `ci` | CI/CD configuration | — |
| `chore` | Maintenance, tooling, configs | — |
| `revert` | Reverts a previous commit | — |

---

## Breaking Changes

Signal a breaking change in TWO ways (either works):

```bash
# Way 1: ! before :
feat!: change commission response shape

# Way 2: BREAKING CHANGE footer
feat: change commission response shape

BREAKING CHANGE: `commission.status` is now an enum string instead of a numeric code
```

If `!` is used, BREAKING CHANGE footer MAY be omitted.

---

## Examples

### Basic commits

```bash
feat: add artist profile page
fix: prevent double submit on commission form
docs: update README with setup instructions
style: format dashboard components with prettier
refactor: extract price calculation logic to utils
test: add unit tests for zod commission schema
chore: bump vite to v8.1
```

### Breaking changes

```bash
feat!: return nested artist object in commission response

BREAKING CHANGE: GET /commissions now returns `{ artist: { id, name } }` instead of `artistName`
```

```bash
fix!: rename `accessToken` to `token` in login response
```

### Multi-line body

```bash
fix: snapshot data missing addon prices

The snapshot was only capturing tier base price, ignoring
selected addons. Now each addon's name, type, and value
are included in the immutable snapshot.

Closes #42
```

---

## What NOT to do

```bash
# ❌ Capitalized type
Fix: typo in login page

# ❌ Period at end of description
feat: add artist profile page.

# ❌ Missing space after colon
feat:missing space

# ❌ Vague description
fix: stuff

# ❌ Too long description (keep under 72 chars)
feat: add the ability for artists to create, edit, and delete tiers with addons and set their base price and estimated delivery days

# ✅ Keep it short, let body explain
feat: add tier CRUD with addons
```

---

## Quick Reference

```
feat: add gallery image upload
fix: recalculate price on addon remove
docs: add API endpoint documentation
style: fix indentation in store files
refactor: split order list into subcomponents
perf: cache public artist profile endpoint
test: add edge cases for price calculator
build: configure path aliases in vite
ci: add lint step to github actions
chore: update dependencies
revert: "feat: add social login"

BREAKING CHANGE: old token format is no longer supported
```
