# Environment Variable Configurations

## What & Why

Secrets (API keys, DB passwords) and environment-specific settings (dev vs staging vs prod URLs) should never be hardcoded into source code. Environment variables are the standard mechanism for injecting this config at runtime instead.

## How It Works

```bash
# .env file (never committed to git — add it to .gitignore)
DATABASE_URL=postgresql://user:pass@localhost/db
OPENAI_API_KEY=sk-...
APP_ENV=development
```

```python
from dotenv import load_dotenv
import os

load_dotenv()                                  # reads .env into process env vars
db_url = os.environ["DATABASE_URL"]             # raises KeyError if missing — good, fail loudly
app_env = os.environ.get("APP_ENV", "development")  # safe default for optional config
```

- **`python-dotenv`**: loads a local `.env` file into `os.environ` for local development — production environments (containers, cloud) set real environment variables directly instead of shipping a `.env` file.
- **Config separation by environment**: the same code reads `DATABASE_URL`, but its *value* differs between dev/staging/prod — the code never branches on environment name for logic, only for which config values apply.

## Pitfalls

- Committing a `.env` file with real secrets to git — even a private repo's history is a leak waiting to happen. Always `.gitignore` it and commit a `.env.example` with placeholder values instead.
- Using `os.environ.get(...)` with a silent default for something that's actually required (like a DB password) — better to let it fail loudly with `os.environ[...]` so a missing secret is caught immediately, not three requests into production.

## Connections

Directly the same secret-handling discipline as Module 4's *Managing GitHub secrets* and Module 3's AWS IAM/credentials — same problem, different storage layer.
