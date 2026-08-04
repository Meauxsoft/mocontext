# Troubleshooting

## The client can't connect

- **Is MoContext running?** Look for the MoContext tray icon, or open
  <http://127.0.0.1:43210/v1/health> in a browser. A JSON response means the
  server is up.
- **Not running?** Start Mo-Search (MoContext starts with it), or launch
  `MoContext.Server.exe` from the Mo-Search install folder
  (`...\Meauxsoft\Mo-Search...\MoContext\`).
- **Port conflict**: MoContext uses port `43210` on `127.0.0.1`. If another
  process holds it, `/v1/health` will fail; free the port and restart.

## Connected, but searches return nothing

- Mo-Search must have indexed the locations you're asking about. Open
  Mo-Search and check which folders/drives are included in the index.
- A brand-new install may still be building its first index — check
  `get_health` → `index.newest_indexed_at` for recency.
- Scoped searches (`working_dir`) return nothing if the directory isn't under
  an indexed root; retry without `working_dir` before concluding a file
  doesn't exist.

## Results look stale

Every search result carries `last_indexed` (when Mo-Search last indexed the
file) and `mtime_newer_than_index` (`true` = the file changed on disk after
indexing). When the flag is `true`, have the agent re-read the file with
`read_file` — it always reads current on-disk content.

## Snippets missing for some files (`content_unavailable`)

- `path_only` — the file type is indexed by path/filename only (binaries,
  media). You get hits, not text.
- `unsupported_loader` — formats like PDF/DOCX are searchable in Mo-Search
  but MoContext does not extract their text for snippets.
- `missing_on_disk` — the file was deleted/moved after indexing.

## Health says "degraded"

One of the two local databases is unreachable. `get_diagnostics` reports the
exact paths. Most common cause: Mo-Search's index database is mid-rebuild —
wait for indexing to finish.

## Still stuck?

Open an issue in this repository with the output of
<http://127.0.0.1:43210/v1/health> (it contains no personal file data).
