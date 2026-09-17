# Shell Scripting Basics

## What & Why

CI/CD pipelines, deployment steps, and quick automation tasks are almost always glued together with small bash scripts. Enough bash to read and write these scripts confidently is a prerequisite for Module 4's CI/CD workflows.

## How It Works

```bash
#!/usr/bin/env bash
set -euo pipefail        # exit on error, undefined var, or failed pipe — always include this

APP_ENV="${1:-development}"   # positional arg with a default value

if [ "$APP_ENV" = "production" ]; then
    echo "Deploying to production..."
else
    echo "Deploying to $APP_ENV..."
fi

for file in ./configs/*.yaml; do
    echo "Found config: $file"
done

echo "Exit code of last command: $?"
```

- **Variables**: `VAR="value"` (no spaces around `=`); referenced as `$VAR` or `${VAR}`.
- **Conditionals**: `if [ condition ]; then ... fi` — note the required spaces inside `[ ]`.
- **Loops**: `for`, `while` — commonly used to iterate over files or retry commands.
- **Exit codes**: every command returns `0` (success) or non-zero (failure); `$?` holds the last one. `set -e` makes the script stop immediately on any non-zero exit.

## Pitfalls

- Omitting `set -euo pipefail` — a script silently continues past a failed command, masking real errors (this bites people constantly in deploy scripts).
- Not quoting variables (`$VAR` instead of `"$VAR"`) — breaks on values containing spaces or special characters.

## Connections

Directly reused inside GitHub Actions workflow steps in Module 4 (*Creating workflow YAML files*, *Triggering automated builds*).
