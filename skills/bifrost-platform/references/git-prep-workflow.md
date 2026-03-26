# Git Prep Workflow

`bifrost` is not responsible for creating GitHub repositories.

Use `git` for repo inspection, commits, and existing remotes.
Use `gh` only when a new GitHub repo must be created from this machine or when GitHub-specific visibility or permission changes are required.

## When to Skip This Workflow

If the repo already has a valid remote that points to the intended source repository, do not recreate it.

## Default Order

1. Inspect repo state.

```bash
git status
git remote -v
```

2. Initialize git only if `.git` is missing.

```bash
git init -b main
```

3. If a remote already exists, reuse it.
4. If the user provides a remote URL, add that with plain `git`.
5. Use `gh repo create` only if the task is explicitly to create a new GitHub repo from this machine.

Example:

```bash
gh repo create <owner>/<repo> --private --source=. --remote=origin
```

6. Commit and push using standard git commands.

```bash
git add <intended-files>
git commit -m "Initial commit"
git push -u origin <branch>
```

7. Only after the remote is valid, run:

```bash
bifrost init ... --json --non-interactive
bifrost deploy ... --json --non-interactive
```

## Commit Hygiene

Before `git add`, avoid accidentally committing:
- `.agents/`
- `.claude/`
- `.codex/`
- editor metadata
- raw `.env*` files unless the user explicitly wants them in git

Be deliberate about whether `.bifrost.yaml` should be committed. It is often useful team context, but it is still a repo-level choice.

## Responsibility Boundary

- `git`: inspect repo state, add remotes, commit, push
- `gh`: optional GitHub repo creation or GitHub-specific changes
- `bifrost`: initialize local platform context, manage infra, deploy, wait, troubleshoot
