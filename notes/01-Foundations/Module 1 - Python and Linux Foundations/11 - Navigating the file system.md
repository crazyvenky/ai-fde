# Navigating the File System

## What & Why

Every deployment script, log inspection, and file-based data pipeline starts with correctly finding and referencing files — and doing it in a way that works the same on your laptop, in CI, and inside a Docker container.

## How It Works

```python
from pathlib import Path

base = Path(__file__).parent          # directory containing this script
data_dir = base / "data"              # path joining with /, not string concat
data_dir.mkdir(parents=True, exist_ok=True)

for file in data_dir.glob("*.pdf"):   # pattern matching
    print(file.resolve())             # absolute path
```

- **`pathlib.Path`** (modern, preferred) vs **`os.path`** (older, string-based) — `pathlib` gives you an object with methods (`.exists()`, `.is_file()`, `.parent`) instead of a pile of separate functions.
- **Absolute path**: full path from the filesystem root (`/home/user/app/data/file.txt`). **Relative path**: relative to the current working directory (`data/file.txt`) — fragile if the working directory changes (very common bug in scripts run from different locations, including cron jobs and CI).

## Pitfalls

- Hardcoding relative paths that only work when the script is run from one specific directory — breaks the moment it's invoked from CI, cron, or a different working directory.
- Using string concatenation (`dir + "/" + filename`) instead of `Path` joining — breaks on Windows and mishandles edge cases like trailing slashes.

## Connections

Basic prerequisite for the shell scripting and environment variable topics later in this module, and for locating config/data files during containerized deployment (Module 4).
