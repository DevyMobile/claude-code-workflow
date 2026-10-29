---
name: commit
description: Commit staged/unstaged changes following a team convention — ticket-ID prefix taken from the branch name (or a conventional-commit type when there is no ticket), no AI co-author trailer. Use for "commit this", "커밋해줘", or any request to commit changes.
---

# Commit

Check changes, show the proposed commit message, then commit. No approval
wait before committing — show the message and proceed.

**Argument**: optional commit message hint.

> This is a template. Replace the placeholders (`PROJ`, message language,
> length limit) with your team's own convention.

## Rules (always apply)

1. **Message language: `<your language>`.** (The original used Korean.)
2. **Never add a `Co-Authored-By` line or any other AI co-author trailer.**
3. **Ticket branch → `[TICKET-ID]` prefix, no conventional-commit type.**
   - `feature/PROJ-123` → prefix `[PROJ-123]`
   - `fix/PROJ-84` → prefix `[PROJ-84]`
   - Branches with no ticket ID (`develop`, `main`, `release/v1.2.0`) get a
     conventional-commit type instead: `feat:`, `fix:`, `chore:`, `docs:`,
     `refactor:`.

## Execution

### Step 1 — Read the change set

Run in parallel:

```bash
git status
git diff --staged
git diff
git log --oneline -5
git branch --show-current
```

- Nothing staged → list unstaged changes and ask which files to include.
- No changes at all → report "nothing to commit" and stop.

### Step 2 — Extract the ticket ID

From the current branch name:

```
feature/PROJ-123  →  [PROJ-123]
fix/PROJ-84       →  [PROJ-84]
develop           →  (none)
release/v1.2.0    →  (none)
```

Regex: `(feature|fix|chore|refactor)/([A-Z]+-[0-9]+)`, use the second capture
group.

### Step 3 — Draft the commit message

Analyze the changed files and diff to draft the message.

- Use the argument (if given) as a hint.
- Ticket ID present → prefix it: `[PROJ-123] message`.
- No ticket ID → prefix a conventional-commit type: `chore: message`.
- Keep it under 50 characters.
- Multiple kinds of changes → summarize around the single main purpose.
- Explain **why** in the body only when the decision is not obvious from the
  diff.

Show it in this format, then go straight to Step 4:

```
Commit message:

  [PROJ-123] Add release-notes workflow
```

### Step 4 — Commit

1. Stage files **explicitly by name** (never `git add .` / `git add -A` — it
   can pull in sensitive files, and in a multi-session setup it can pull in
   files another session staged).
2. Stage immediately before committing, not earlier. Do not leave anything
   staged while waiting for an answer.
3. Run `git commit -m "message"`.
4. Print the commit hash and message.
