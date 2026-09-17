# Permission and User Management

## What & Why

Every server this course deploys to runs Linux under the hood. Misunderstanding file permissions and users is a top cause of both "why can't my app write this file" bugs and real security incidents (running services as root when they don't need to be).

## How It Works

```
-rwxr-xr--  1 appuser  appgroup  1024  Jan 1 12:00  deploy.sh
 │└┬┘└┬┘└┬┘
 │ │  │   └─ others: r-- (read only)
 │ │  └───── group:  r-x (read + execute)
 │ └──────── owner:  rwx (read + write + execute)
 └────────── file type (- = regular file, d = directory)
```

- **`chmod`**: changes permission bits (`chmod 755 file` = owner rwx, group r-x, others r-x).
- **`chown user:group file`**: changes file ownership.
- **Users and groups**: every process runs as a specific user; that user's permissions (plus their group memberships) determine what it can read/write/execute.
- **`sudo` vs root**: `sudo` runs a *single command* with elevated privileges and logs who did it; being logged in *as* root has no such audit trail and is far riskier.

## Pitfalls

- Running an application process as root "to avoid permission errors" instead of fixing the actual permission — massively increases blast radius if the app is ever compromised.
- `chmod 777`-ing a file/directory as a quick fix — grants write+execute to *everyone*, including other users/processes on the same machine.

## Connections

The same least-privilege thinking reappears as IAM policies in Module 3 (*Principle of least privilege*, *Creating identity policies*) and as database roles in Module 11 (*Implementing read-only database roles*) — same principle, different layer.
