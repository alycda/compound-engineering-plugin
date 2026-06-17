# Committing with Jujutsu (jj)

This repository uses Jujutsu. Follow this workflow for the rest of the skill instead of the git steps in `SKILL.md`. The goal is unchanged -- one or two well-crafted, convention-following commits from the current changes -- but jj's model differs from git in ways that change the mechanics.

## How jj differs from git

- **No staging area.** Every change in the working copy is already part of the working-copy commit, addressed as `@`. There is no `git add`; select files at commit time with filesets instead.
- **The working copy is a commit.** `@` is an ordinary (usually undescribed) commit. "Committing" means giving `@` a description, and optionally finalizing it and starting a fresh change on top.
- **No detached-HEAD or accidental-default-branch footgun.** Creating a commit does not move any bookmark (jj's name for a branch), so there is no "committed to main by mistake" case and no need to create a feature branch first. Bookmark and push management are deferred to push time and are out of scope for this commit-only skill -- do not create bookmarks here.

## Step 1: Gather context

Run these, one command per call:

```bash
jj status
```

```bash
jj diff
```

```bash
jj log --limit 10
```

`jj status` shows the working-copy changes and the current `@`. `jj diff` shows the full diff of `@`. `jj log --limit 10` gives recent history for convention detection.

If `jj status` reports no changes in the working copy, there is nothing to commit -- report that and stop.

## Step 2: Determine commit message convention

Same priority order as the git path:

1. **Repo conventions already in context** -- project instructions (AGENTS.md, CLAUDE.md) loaded at session start. Follow them if they specify a convention.
2. **Recent commit history** -- examine the messages from `jj log`; if a clear pattern emerges (conventional commits, ticket prefixes, emoji prefixes), match it.
3. **Default: conventional commits** -- `type(scope): description` with `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `ci`, `style`, `build`.

Where `fix:` and `feat:` both seem to fit, default to `fix:` -- a change that remedies broken or missing behavior is `fix:` even when implemented by adding code. Reserve `feat:` for capabilities the user could not previously accomplish. The user may override.

## Step 3: Consider logical commits

Scan the changed files for naturally distinct concerns, exactly as the git path does. Group at the **file level only** -- do not split hunks within a file. Two or three logical commits is the sweet spot; when the split is ambiguous, one commit is fine.

jj automatically snapshots all non-ignored working-copy files into `@`. There is no `git add -A` to avoid, but the same caution applies: confirm that sensitive or generated files (`.env`, credentials, build artifacts) are covered by `.gitignore` so they are not part of `@`. If `jj status` shows such a file as added, stop and surface it before committing rather than baking it into history.

## Step 4: Describe and commit

**Single logical commit** -- describe `@` in place. Use a quoted heredoc so multi-line bodies keep their formatting:

```bash
jj describe -m "$(cat <<'EOF'
type(scope): subject line here

Optional body explaining why this change was made,
not just what changed.
EOF
)"
```

**Multiple logical commits** -- for each group except the last, finalize just that group's files into a commit and leave the rest in the working copy:

```bash
jj commit file1 file2 -m "$(cat <<'EOF'
type(scope): first group subject
EOF
)"
```

`jj commit <filesets>` moves only the named paths into a finalized commit (now `@-`) and keeps the remaining changes in the new working-copy commit `@`. Repeat for each intermediate group, then describe the final remaining group in place with `jj describe -m` as above. Naming files explicitly is the jj analog of staging files by name -- it keeps unrelated files out of a group.

Write subject lines in imperative mood, focused on *why* not *what*. Add a body (blank line after the subject) for non-trivial changes; omit it for obvious single-purpose ones.

## Step 5: Confirm

```bash
jj log --limit 5
```

Report the resulting commit(s) -- change ID and subject line for each.
