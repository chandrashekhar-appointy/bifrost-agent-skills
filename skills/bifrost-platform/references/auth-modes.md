# Auth Modes

Choose auth mode based on who is running the command and where it runs.

## 1. Personal Access Token

Use for:
- AI agents
- CI/CD
- headless automation

Preferred examples:

```bash
bifrost auth login --token <bf_pat_...> --json --non-interactive
bifrost auth whoami --json --non-interactive
```

Environment-variable alternative:

```bash
BIFROST_TOKEN=<bf_pat_...> bifrost auth whoami --json --non-interactive
```

Prefer PATs for automation because they avoid browser/device interaction.

## 2. Device Flow

Use for:
- humans on remote Linux hosts
- SSH sessions without a browser
- environments where opening a browser is impractical

Example:

```bash
bifrost auth login --headless --json --non-interactive
```

The CLI prints the verification URL and code to stderr and polls until approval completes.

## 3. Browser Login

Use for:
- local desktop users with a browser

Example:

```bash
bifrost auth login --json --non-interactive
```

This flow opens or coordinates with a browser-based login path.

## Verification Step

After any login flow, verify identity:

```bash
bifrost auth whoami --json --non-interactive
```
