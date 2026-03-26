# Git Prep Workflow

`bifrost` is not responsible for creating GitHub repositories.

Use `git` and `gh` to bootstrap the remote when needed, then return to the Bifrost CLI.

## When to Skip This Workflow

If the repo already has a valid remote that points to the intended source repository, do not recreate it.

## When No Remote Exists

1. Check repo state.

```bash
git status
```

2. Initialize git if needed.

```bash
git init
```

3. Create the remote repository with GitHub CLI.

```bash
gh repo create <owner>/<repo> --private --source=. --remote=origin
```

4. Commit and push using standard git commands.

```bash
git add .
git commit -m "Initial commit"
git push -u origin <branch>
```

5. Only after the remote is valid, run:

```bash
bifrost init ... --json --non-interactive
bifrost deploy ... --json --non-interactive
```

## Responsibility Boundary

- `git` and `gh`: initialize repo, create remote, push code
- `bifrost`: initialize local platform context, manage infra, deploy, wait, troubleshoot
