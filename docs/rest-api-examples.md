# REST API examples

Every MCP tool is also a plain HTTP endpoint on `http://127.0.0.1:43210` —
useful for scripts, curl, or clients without MCP support. JSON fields use
`snake_case`.

MoContext binds to `127.0.0.1`, so these endpoints are reachable only from the
same computer and there is no remote surface. There is no authentication token
configured by default, which means any process running as your Windows user can
call them — treat local software and AI clients as part of the same trust
boundary. See the [security policy](../SECURITY.md) before considering any
arrangement that exposes the port beyond loopback.

## Health and orientation

```powershell
# Liveness + index recency
Invoke-RestMethod http://127.0.0.1:43210/v1/health

# Full orientation: capabilities, budgets, doc/memory paths
Invoke-RestMethod http://127.0.0.1:43210/v1/server-summary
```

## Find files

```powershell
# Ranked full-text search
Invoke-RestMethod "http://127.0.0.1:43210/v1/context/find-files?query=smtp+settings&max_files=10"

# Recently indexed files (no query)
Invoke-RestMethod "http://127.0.0.1:43210/v1/context/find-files?max_files=20&path_filter=Projects"

# Hot/warm working set: what changed in the last 24h / 7d
Invoke-RestMethod "http://127.0.0.1:43210/v1/context/find-files?hot_hours=24&warm_days=7"
```

## Evidence pack (search with source snippets)

```powershell
Invoke-RestMethod -Method Post http://127.0.0.1:43210/v1/context/evidence-pack `
  -ContentType 'application/json' `
  -Body (@{
    query     = 'session token validation'
    mode      = 'keyword'   # keyword | phrase | all
    max_files = 5
  } | ConvertTo-Json)
```

Each returned file includes ranked windows with line numbers, plus
`last_indexed` and `mtime_newer_than_index` freshness fields.

## Read a file

```powershell
Invoke-RestMethod -Method Post http://127.0.0.1:43210/v1/context/read-file `
  -ContentType 'application/json' `
  -Body (@{ path = 'C:\Projects\app\Program.cs'; start_line = 1; end_line = 80 } | ConvertTo-Json)
```

## Activity journal

```powershell
# Save a checkpoint note
Invoke-RestMethod -Method Post http://127.0.0.1:43210/v1/activity `
  -ContentType 'application/json' `
  -Body (@{
    summary_text  = 'Refactored JWT validation into AuthService.'
    topic_key     = 'auth-refactor'
    next_step_text = 'Wire AuthService into the login endpoint.'
  } | ConvertTo-Json)

# Recent notes, grouped by topic
Invoke-RestMethod "http://127.0.0.1:43210/v1/activity/recent?group_by=topic"
```

## Long-term memory digests

```powershell
# Load recent saved digests + working-set hints
Invoke-RestMethod http://127.0.0.1:43210/v1/context/digest

# Search digests by keyword
Invoke-RestMethod "http://127.0.0.1:43210/v1/context/digest?q=migration"
```

For the full endpoint list and parameters, call `/v1/server-summary` or see
the [AI usage guide](ai-usage.md).
