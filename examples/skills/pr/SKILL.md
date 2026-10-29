---
name: pr
description: Review the current branch's diff against its base, reconcile the project's own CLAUDE.md if it has gone stale, then open a PR — proceeding through to PR creation unless the review finds a real problem. Use for "open a PR", "PR 만들어줘", or any request to create a pull request.
---

# PR

Works the same across any stack — it never hardcodes per-stack rules.
Whatever the project's own CLAUDE.md documents is what gets checked against
the diff.

**Argument**: optional base branch name (default: auto-detect).

> This is a template. Replace the placeholders (base-branch preference, label
> names, PR language) with your team's own convention.

## Approval policy

Only stop and ask the user when the review in Step 3 finds an actual
problem — that is the one decision only the user can make (fix now vs. ship
anyway). Every other step (CLAUDE.md reconciliation, PR content proposal,
push, PR creation) proceeds without waiting for confirmation.

## Execution

### Step 1 — Gather branch info

```bash
git branch --show-current
git log --oneline origin/develop..HEAD 2>/dev/null || git log --oneline origin/main..HEAD 2>/dev/null
git diff origin/develop...HEAD --stat 2>/dev/null || git diff origin/main...HEAD --stat 2>/dev/null
```

- Argument given → use that branch as base.
- No argument → prefer `origin/develop`, fall back to `origin/main`.
- Extract the branch prefix (`fix/`, `feature/`, `refactor/`, ...) from the
  current branch name.

### Step 2 — Reconcile CLAUDE.md, commit separately

```bash
git diff origin/develop...HEAD --name-only 2>/dev/null || git diff origin/main...HEAD --name-only 2>/dev/null
```

1. Read the project's `CLAUDE.md` at the repo root.
2. Compare the changed-files list and diff against what CLAUDE.md documents
   (architecture, module list, versions, directory structure, ...). Don't
   assume any particular stack; just check whether this diff makes an
   existing section stale.
3. If something is stale, edit CLAUDE.md to match reality.
4. If CLAUDE.md changed, commit it now, **separately from the code
   commits**, staging only that file immediately before the commit:
   ```bash
   git add CLAUDE.md
   git commit -m "docs: update CLAUDE.md"
   ```
5. If nothing needs updating, skip silently — don't report "no changes."

### Step 3 — Inline review

Review directly instead of invoking a separate review skill:

1. Get the full diff: `git diff origin/develop...HEAD` (or
   `origin/main...HEAD`).
2. Read the changed files for context.
3. Check only for:
   - Clear bugs: wrong conditions, null dereferences, missing error
     handling, wrong variable used.
   - Removed behavior: whether a deleted guard/validation is still
     guaranteed by the new code.
   - Call-site impact: whether a changed function signature breaks a
     caller.

### Step 4 — Conditional gate

- **Problems found** → stop. Report them (`file:line`, one line each) and
  ask whether to fix first or proceed anyway. Wait for the answer.
- **No problems** → report "review clean" and continue straight to Step 5.

### Step 5 — Propose PR content

```bash
gh label list --limit 50
```

Map the branch prefix to a label from what actually exists:
- `fix/` → `Bug` or `Fix`
- `feature/` → `Feature`
- `refactor/` → `Refactor` or `Enhancement`
- No match → omit the label.

Draft the PR title (include the ticket ID if the branch has one, e.g.
`PROJ-130`) and body from the commits and diff:

```
## Proposed PR

**Title**: [PROJ-XXX] title
**Assignee**: @me
**Label**: Feature / Bug / Fix / ...

**Body**:
---
## What changed
- ...

## Why
- ...

## Test plan
- [ ] ...
---
```

Show this, then proceed straight to Step 6.

### Step 6 — Create the PR

```bash
git push -u origin HEAD
gh pr create \
  --title "title" \
  --body "$(cat <<'EOF'
body
EOF
)" \
  --assignee @me \
  --label "label" \
  --base <base-branch>
```

Omit `--label` if none matched. Print the created PR URL to close out.
Do **not** merge the PR.
