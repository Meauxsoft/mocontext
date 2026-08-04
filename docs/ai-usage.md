# MoContext AI Usage Guide - MCP Edition

## What Is MoContext?

MoContext is a local Model Context Protocol (MCP) server that runs alongside MoSearch. It gives you several capabilities:

1. **Evidence-pack retrieval** - ranked, budget-capped source code snippets drawn directly from the MoSearch full-text index (`MoSl.db`). This is grounded evidence you can trust.
2. **File discovery** - one tool (`find_files`) covers keyword search, recently indexed browsing, and the hot/warm recently-active working set.
3. **Activity journal** - lightweight continuity notes you write across sessions, stored in a separate `MoContext.db`. These are *hints*, not ground truth.
4. **Long-term context digests** - explicit `SaveContext` / `LoadContext` records saved as human-editable Markdown under `MoContext\weeklyDigest`.
5. **Indexed context files** - installed MoContext docs plus user/AI-authored Markdown memory files under the per-user MoContext folder. These are discoverable through MoSearch after the folder is indexed.

**Mental model:** source snippets from the index are **evidence**. Working-set rows are **recent activity hints**. Activity notes, context digests, and memory files are **durable orientation**, not proof. Never trust a note, digest, or memory file over re-checking the actual source file.

---

## Connecting

Configure your MCP client to connect to:

```
http://127.0.0.1:43210/mcp
```

Tools, resources, and resource templates are auto-discovered by MCP clients. No manual endpoint wiring is needed when the client supports MCP.

If the client exposes MoContext resources, useful entry points are:

- `mocontext://server/summary` - concise status, purpose, endpoint, and first-call guidance.
- `mocontext://usage-guide` - compact guidance for choosing MoContext tools/resources.
- `mocontext://context-files` - inventory of root-level transition docs and user/AI-owned memory files.
- `mocontext://recent-activity` - recent continuity notes with default limits.
- `mocontext://docs/ai-usage` - this installed AI usage guide.
- `mocontext://docs/human-prompt` - the human-facing session starter prompt.
- `mocontext://docs/human` - human guide with copyable custom-instructions/session prompts and a technical appendix.
- `mocontext://memory/readme` - guidance for durable MoContext memory files.

If the client exposes only tools, `get_server_summary` plus `get_usage_guide` give the same high-level orientation.

---

## Verifying the Server

Call `get_server_summary` first. It returns the whole front door in one call: server status, version, HTTP/MCP endpoints, data-source readiness, suggested next calls, **plus embedded `capabilities`** (budgets, search modes, recommended flows) **and `context_paths`** (sys-doc and memory folder locations). There is no separate capabilities or context-paths tool.

- **`context_paths.system_docs_path`** - product-owned docs seeded by MoContext/installer. Do not treat this as user memory.
- **`context_paths.memory_path`** - user/AI-owned Markdown notes intended for durable, editable long-term context.

Call `get_health` for a small liveness/readiness check. A healthy response has `status: "ok"` and includes:

- **`databases.mo_search_readable`** - confirms the MoSearch full-text index is accessible.
- **`databases.mo_context_writable`** - confirms the activity journal DB is writable.
- **`index.indexed_extension_count`** - confirms the indexed-extension profile is readable without returning the full inventory.
- **`index.newest_indexed_at`** - ISO-8601 UTC time of the most recent index event across the whole index. If this is hours or days old, the indexing service may be paused or falling behind; per-file `last_indexed` values can be no newer than this.
- **`warnings`** - human-readable degraded-state hints.

If `status` is `"degraded"`, a database is inaccessible. If the tool call fails entirely, MoContext is not running - the user needs to start it.

Call `get_diagnostics` only when you need verbose details such as runtime paths, deployment provenance, database paths, and the full `indexed_extensions` inventory. Files with extensions not in this list may match by path/filename only - you get hit counts but no snippets (`content_unavailable: "path_only"`).

Call `list_context_files` to see what MoContext can read from its context folder without guessing paths. Use `read_context_file` with the returned `scope` and `relative_path`. Use `write_memory_file` only for small `.md` or `.txt` notes under `memory`; it will not write to `sys` or arbitrary file paths.

MoContext uses `snake_case` for HTTP JSON fields, HTTP query/body names, MCP tool arguments, MCP tool results, and JSON resources. MCP protocol envelope fields such as `protocolVersion`, `serverInfo`, and `mimeType` remain whatever the MCP spec requires.

---

## Sessions

There is no session-start tool. `record_activity` auto-generates a shared ad-hoc session id when you omit `session_id` and returns it in the response. Pass an optional `client_name` (e.g. `"Cascade"`) to fold your client into the ad-hoc id (`sess-adhoc-cascade`) so multi-client machines can tell journals apart. You can also pass any explicit `session_id` of your own; unknown ids are auto-created.

Sessions are optional for read-only retrieval - they only matter if you record activity notes.

---

## Compaction Checkpoints

When an AI client compacts its context window (replacing earlier conversation
with a summary), detail is lost that MoContext can preserve. Compaction is an
infrequent but significant event - treat it as an automatic checkpoint
trigger.

**The rule for agents:** if you notice your context was just compacted (a
continuation summary stands in for earlier turns), call `record_activity`
before resuming work:

- `summary_text` - what has been completed so far, concrete: files touched,
  commits made, decisions taken. State, not narration.
- `next_step_text` - the exact next action.
- `topic_key` - the ongoing thread, so entries group across sessions.

Keep it to one journal note per compaction. Do **not** auto-save a context
digest: repeated compactions in a long session would produce near-duplicate
digests, and the weekly rollup already consolidates journal entries. Digests
remain reserved for the `SaveContext` trigger word and explicit user requests.

**Deterministic setup for Claude Code:** its `SessionStart` hook fires with
matcher `compact` after every compaction, and the hook's stdout is injected
into the agent's context. Add this to `.claude/settings.json` (project) or
`~/.claude/settings.json` (user):

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "compact",
        "hooks": [
          {
            "type": "command",
            "command": "echo Context was just compacted. Before resuming, call the MoContext record_activity tool: summary_text = concrete work completed so far (files, commits, decisions), next_step_text = the exact next action, topic_key = the ongoing thread. Then continue the task."
          }
        ]
      }
    ]
  }
}
```

Other MCP clients have no standard compaction signal; there the
notice-it-yourself rule above is the fallback, and it is included in the
installed Human Prompt.

---

## Workflow 1 - Finding Source Code Evidence

Use `get_evidence_pack`. This is the primary retrieval tool - call it whenever you need grounded source evidence before making a factual claim.

**Key parameters:**
- **`query`** (required) - search terms.
- **`mode`** - `"keyword"` (default, OR), `"all"` (AND), or `"phrase"` (literal match).
- **`working_dir`** - your current working directory (absolute path). Recommended on every retrieval call; results are scoped to files under it and the response echoes `scope_applied`. Omit it to search all indexed roots - do this when scoped results are empty or the user asks about files outside the project. Also supported by `find_files`.
- **`path_filter`** - substring filter on file paths (e.g. `"mobase"`).
- **`filename`** - substring filter on filenames (e.g. `".cpp"` or `"Program"`).
- **`max_files`**, **`max_windows_per_file`**, **`context_lines`** - budget controls.

**Search mode guidance:**
- **`keyword`** - almost always the right starting point. OR semantics across terms.
- **`all`** - every term must appear in the same file. Very restrictive. Use only when you know several identifiers co-occur.
- **`phrase`** - literal string match. Best for error messages or exact fragments. Note: the tokenizer splits on dots and hyphens, so `MoContext.Server.exe` becomes three tokens. Use `keyword` for dotted identifiers.

**Best practice:** Pass `working_dir`, then combine `path_filter` and `filename` to narrow further. Without scoping, common terms match across the entire indexed codebase.

**Ranking note:** Path-only files (binaries, media) are scored on recency and hit share, not text density, so readable files rank ahead of build artifacts in typical queries.

**Interpreting the response:**
- **`files_truncated: true`** - more files matched than returned. Narrow with `path_filter` or fewer terms.
- **`lines_truncated: true`** - line budget hit. Reduce `max_files`, increase `context_lines`.
- **`content_unavailable: "path_only"`** - file is indexed by path only (e.g. `.iss`, `.rc`). You get path and hit count but no snippets.
- **`content_unavailable: "unsupported_loader"`** - binary format (PDF, DOCX). Same.
- **`ranking.ai_score`** - hit density weighted by recency. Higher = more relevant.
- **`ranking.effective_loader`** - `0` = path-only, `100` = full-text indexed.
- **`scope_applied`** - the working-directory scope that filtered this response, or null. If a search returns nothing and `scope_applied` is set, retry without `working_dir` before concluding the code doesn't exist.
- **`last_indexed`** - ISO-8601 UTC time MoSearch last indexed this file, or null when unknown.
- **`mtime_newer_than_index`** - `true` means the file changed on disk after it was last indexed, so ranking and snippets may reflect stale content; `false` means the index is current for this file; `null` means the comparison could not be made (file missing or unreadable).

**Freshness:** check `last_indexed` and `mtime_newer_than_index` per result instead of blanket-distrusting the index. When `mtime_newer_than_index` is `true` (or `null`), re-read the file from disk with `read_file` before making factual claims; when it is `false`, the returned snippets match the current on-disk content.

---

## Workflow 2 - Finding Files (search, browse, working set)

Use `find_files` - the one tool for file discovery. Its behavior follows from the parameters you pass:

- **With `query`** - ranked keyword/phrase search against the MoSearch index. Returns file paths with rank; no snippets. Good for choosing which file to drill into before `get_evidence_pack` or `read_file`.
- **Without `query`** - lists recently indexed files in index-recency order. Good for session-start project discovery; no keywords required.
- **With `hot_hours` and/or `warm_days` (and no `query`)** - returns the `UriHistory` working-set grouping instead: `hot` (recently indexed within `hot_hours`, default 24) and `warm` (within `warm_days`, default 7 days, deduped against hot). Entries carry path, change count, first/last indexed time, size, and mod generation - metadata only, never file contents.

All three forms accept `working_dir`, `path_filter`, `filename`, and `max_files`. The response's `mode` field says which form was returned (`search`, `browse`, or `working_set`). Search and browse entries carry the same `last_indexed` / `mtime_newer_than_index` freshness fields as evidence-pack results (Workflow 1); working-set entries carry their own first/last indexed times.

Default `LoadContext` includes the same working-set metadata so a new chat can see active files without loading them. `LoadContext <keyword>` stays focused on matching saved digests and does not include the working set by default.

Treat returned paths as candidates for `read_file` or `get_evidence_pack`, not as automatically important.

---

## Workflow 3 - Reading Files

Use `read_file`. One tool, three addressing forms:

- **`path` alone** - the whole file (subject to the server line budget; `truncated: true` if capped).
- **`path` + `start_line`/`end_line`** - an arbitrary line range.
- **`path` + `line`** (with optional `before`/`after` context radii) - a focused window around one line. Best for follow-up drilling after evidence-pack identifies a region.

---

## Workflow 4 - Recording Continuity Notes

Use `record_activity` to save a short note after a **meaningful** work segment completes.

**Note discipline:**
- **`summary_text`** - what was just accomplished. One to three sentences. Max 500 chars.
- **`next_step_text`** - the single most important next action. One sentence.
- **`query_hint`** - an optional search query for future retrieval relevance.
- **`topic_key`** - a short slug like `avl-tree-refactor` or `installer-prep`. Used for filtering. Vary across work areas - if everything uses the same key, the briefing becomes repetitive.
- **`client_name`** - optional client identifier folded into the ad-hoc session id when `session_id` is omitted.

**Do not call this after every tool invocation.** Only at significant milestones.

---

## Workflow 5 - Recovering Prior Context

Call `get_activity_briefing` **once at the start of a new session** to recover a compact plain-text summary of recent activity. The `briefing` string is ready to inject directly into your context.

Use `get_recent_activity` for full structured detail (filterable by session, topic, keyword). Pass `group_by: "topic"` to get the active-topic overview instead - topic keys with entry counts, useful when you don't remember the exact slug.

**Important:** The briefing gives hints about prior work, not verified facts. Always re-retrieve source evidence before making claims based on a briefing.

---

## Workflow 6 - Durable Indexed Memory

`get_server_summary`'s `context_paths` gives you the `memory_path`. Files in this folder are intended for durable, human-editable context that should be searchable by MoSearch and retrievable by MoContext after indexing.

Good memory-file content:

- Project maps and glossary-style orientation.
- Durable design decisions and human corrections.
- Short topic summaries that would help a future AI session.

Avoid secrets, raw chat logs, and noisy per-step changelogs.

When looking for memory, start with `list_context_files` or `mocontext://context-files`. Then read selected entries with `read_context_file`. If you need indexed discovery by topic, use `find_files` or `get_evidence_pack` with a `path_filter` such as `MoContext\memory`.

Only create or update memory files when it would clearly help future sessions or the user asks for it. Prefer short Markdown files organized by project/topic. Use `write_memory_file` with a relative path like `projects/MoSearch_building.md`; pass `overwrite: true` only after reading the existing file and intentionally replacing it. Humans may edit these files directly; treat their edits as stronger than prior AI-authored text, but still re-check source before factual claims.

---

## Workflow 7 - SaveContext / LoadContext Long-Term Memory

This is the MoContext successor to the older WindCache `DoSave` / `DoLoad` habit. WindCache proved that explicit context save/load makes AI sessions faster and less forgetful. MoContext keeps the human-facing verbs but moves the mechanism to HTTP/MCP tools, Markdown digest files, MoSearch-backed search, and live index metadata.

Use these tools only when the user triggers them or has explicitly configured an automatic rule:

- **`SaveContext`** - call `save_context_digest` with the schema-v2 fields. Be dense: enumerate concrete changes and touched files, give every decision a rationale, and record gotchas as typed findings. Do not include secrets, tokens, credentials, or raw chat logs.

  ```json
  {
    "summary": "2-5 sentences: what this session accomplished and where it left off.",
    "major_work": [
      {
        "feature": "Short feature/task name",
        "description": "What changed and why",
        "changes": ["concrete change 1", "concrete change 2"],
        "files_created": ["path"],
        "files_modified": ["path"],
        "status": "done | in-progress | blocked"
      }
    ],
    "decisions_v2": [
      { "decision": "What was decided", "rationale": "Why - the part future sessions need" }
    ],
    "findings": [
      { "type": "gotcha | pattern | preference | decision", "text": "...", "confidence": "confirmed | tentative" }
    ],
    "pending_work": ["unfinished thread / next step"],
    "tags": [], "files": [], "open_questions": [], "session_id": null
  }
  ```

  Legacy plain-string `decisions` are still accepted and lifted into `decisions_v2` with an empty rationale.

  **Quality gate**: the server rejects digests that have a summary under 200 characters AND no `major_work` AND no decisions. A rejection error means the digest was too thin to be useful long-term memory - expand the summary or add structured entries, then retry the save. The rejection is not a transient failure.
- **`LoadContext`** - call `load_context_digest` with no `q`. It returns recent saved digest records from the last two weeks plus a compact `working_set` hint from `UriHistory`. Responses are budget-capped; if `truncated` is true, check `omitted_records`.
- **`LoadContext <keyword>`** - call `load_context_digest` with `q=<keyword>`. It searches saved digest Markdown through MoSearch and returns matches/snippets. Call `load_context_digest` again with `id=<match id>` to fetch a full matched record when needed. Responses are budget-capped; if `truncated` is true, check `omitted_matches`.

If the user asks when to save: recommend typing `SaveContext` after a commit, at a milestone, or before ending a session — the WindCache `DoSave` habit.

Saved digests live under `MoContext\weeklyDigest\<ISO-week>\` as one Markdown file per save. Humans may edit them directly; MoSearch will index the changed text. Treat digest content as durable orientation, not proof. Use it to decide what source files or docs to inspect next, then verify.

Each week folder also contains an auto-generated `_WeekSummary.md` rollup: a weekly narrative built from the activity journal plus saved digests, regenerated at startup, after every digest save, and hourly. Its banner says "Auto-generated by MoContext - hand edits will be lost"; never hand-edit it or write to it - durable notes belong in SaveContext digests or memory files. It is indexed by MoSearch, so `LoadContext <keyword>` can match weekly narratives, but it is not a digest record.

The working-set section in default `LoadContext` is metadata only. It helps identify recently edited/re-indexed files, but a frequently indexed path might still be generated, noisy, or irrelevant. Do not automatically load every working-set file.

`LoadContext` accepts `max_chars` when a smaller response is needed. The server also enforces a hard 256KB ceiling regardless of caller input.

---

## Complete Tool Reference

All 15 MCP tools:

| Tool | Description |
|------|-------------|
| `get_server_summary` | One-call orientation: status, endpoints, data sources, purpose, suggested next calls, plus embedded `capabilities` (budgets, modes, flows) and `context_paths` (sys-doc + memory folder paths). |
| `get_usage_guide` | Compact guidance for choosing MoContext tools and resources. |
| `get_health` | Small liveness/readiness check; `index.newest_indexed_at` shows overall index recency. |
| `get_diagnostics` | Verbose runtime, deployment, database path, and indexed-extension details. |
| `find_files` | File discovery: `query` = ranked search; no query = recently indexed browse; `hot_hours`/`warm_days` = `UriHistory` hot/warm working set. Paths and metadata only, no snippets. |
| `get_evidence_pack` | Primary retrieval: ranked files + source windows. Results carry `last_indexed` / `mtime_newer_than_index` freshness fields. |
| `read_file` | Read a file: whole file, `start_line`/`end_line` range, or a window around `line` with `before`/`after`. |
| `record_activity` | Save an activity note at a milestone; optional `client_name` folds into the ad-hoc session id. |
| `get_recent_activity` | List recent notes, filterable; `group_by: "topic"` returns the active-topic overview. |
| `get_activity_briefing` | Compact plain-text briefing for context injection. |
| `list_context_files` | Inventory root-level transition docs, memory files, and optionally sys docs. |
| `read_context_file` | Read a scoped context file by `scope` and `relative_path`. |
| `write_memory_file` | Create or update a guarded `.md`/`.txt` file under the memory folder. |
| `save_context_digest` | Save a `SaveContext` long-term memory digest as Markdown. Schema v2: `major_work` (changes + touched files), `decisions_v2` (decision + rationale), typed `findings`, `pending_work`. Thin digests (summary < 200 chars, no major_work, no decisions) are rejected with a retry instruction. |
| `load_context_digest` | Load recent digests, search digests by keyword with `q`, or fetch one full record with `id`; default load includes working-set hints. Budget-capped; reports `truncated`, `omitted_records`, and `omitted_matches`. |

Saved-pack and session-start capabilities were removed from the MCP surface in the 2026-07 redesign; their REST routes remain for one release. Use `record_activity` with a `query_hint` instead of saved packs.

---

## Recommended Session Pattern

```
1. get_server_summary        - one call: status, budgets, modes, sys-doc/memory paths
2. list_context_files        - discover readable docs and durable memory files
3. get_activity_briefing     - recover prior context hints when helpful
4. load_context_digest       - only when the user says LoadContext or explicitly enabled it
5. ... do your work ...
   find_files                - discover files (search / browse / working set)
   get_evidence_pack         - retrieve evidence before factual claims; pass working_dir
   read_file                 - read a complete file, range, or focused window
6. record_activity           - record a note at a meaningful checkpoint
7. save_context_digest       - only when the user says SaveContext or explicitly asks
8. write_memory_file         - update durable memory only when it is worth preserving
9. (repeat 5-8 as needed)
```

Use `get_diagnostics` only when you need the full indexed-extension list or deployment details. Use `get_usage_guide` when you want per-tool guidance without re-reading this document.

---

## Budget Defaults

Check `get_server_summary`'s `capabilities` for current values. Typical defaults:

| Setting | Default | Notes |
|---------|---------|-------|
| `max_files` | 10 | Files per evidence-pack response |
| `max_windows_per_file` | 3 | Snippet windows per file |
| `context_lines_before` | 10 | Lines before each hit |
| `context_lines_after` | 10 | Lines after each hit |
| `max_total_lines` | 400 | Hard line cap per response |
| `max_total_chars` | 20000 | Hard char cap per response |
| `context_digest.max_load_chars` | 20000 | Default `LoadContext` digest/snippet budget |
| `context_digest.hard_max_load_chars` | 262144 | Absolute `LoadContext` cap regardless of caller |
| `max_activity_text_chars` | 500 | Max `summary_text` length |
| `max_recent_activity_days` | 14 | Default lookback for recent/briefing |

You can request fewer than the defaults but not more.

---

## Limitations

- Cannot read PDF, DOCX, XLS, or other binary formats - returns `content_unavailable: "unsupported_loader"`.
- Cannot read certain script formats (`.iss`, `.rc`) that MoSearch indexes by path only - returns `content_unavailable: "path_only"`.
- No semantic / embedding-based search - retrieval is keyword/phrase/AND against the MoSearch word index.
- Tokenizer splits on dots - `MoContext.Server.exe` is three words. Use `keyword` mode for dotted identifiers.
- Cannot search files never indexed by MoSearch.
- `LoadContext <keyword>` depends on MoSearch indexing the digest Markdown; immediately saved or hand-edited digests may need a brief AutoIndexer moment before keyword search sees them.
- `LoadContext` responses may be truncated by `max_chars`; use omitted counts to decide whether to narrow by keyword or fetch a specific digest by id.
- `find_files` working-set mode and the default `LoadContext` working set expose index-event metadata, not semantic importance.
- Loopback only (`127.0.0.1`).

If a file returns `content_unavailable`, note the path and let the user know the content could not be shown directly.

---

## Dangerous Patterns to Avoid

- **Trusting activity notes without re-checking source.** Notes are written by a prior AI session that may have been wrong. Always retrieve fresh evidence before making factual claims.
- **Treating `SaveContext` digests as evidence.** Digests are useful memory, not source-of-truth. Let them orient your search, then inspect real files.
- **Calling `SaveContext` / `LoadContext` proactively without a rule.** Use these only when the user types the trigger or explicitly enables automatic behavior.
- **Putting secrets into digests or memory.** Never summarize tokens, credentials, keys, private personal data, or raw proprietary transcripts into saved context.
- **Reading every working-set file automatically.** The working set is a compact path hint; choose follow-up reads deliberately.
- **Treating memory files as proof.** Memory files are durable orientation, not evidence. Let them guide what to inspect, then verify against source.
- **Writing into `sys`.** `sys` is product-owned and may be refreshed. Put AI/human-authored long-term notes in `memory`.
- **Saving a note after every single tool call.** Notes are valuable only if sparse and meaningful. Save at work segment boundaries, not after every search.
- **Writing long `summary_text` entries.** Keep to two or three sentences. Long notes become a contradiction risk.
- **Using evidence-pack snippets as a substitute for reading the whole file.** Snippets are windowed. The hit region may not contain the full context needed to understand a bug or design choice.

---

## Appendix - User-Side Global Rule

Paste this into an AI client's persistent custom instructions when you want the client to know about MoContext long-term memory:

> **Long-term memory via MoContext.** When the user types `SaveContext`, summarize the current conversation in four short sections: what we did, files touched, decisions, and open questions. Call `save_context_digest` with that summary plus structured `major_work`, `decisions_v2`, `findings`, and `pending_work` when applicable. When the user types `LoadContext`, call `load_context_digest` with no arguments. When the user types `LoadContext <keyword>`, call `load_context_digest` with `q=<keyword>`. Fold returned digests into working context silently; do not repeat them verbatim unless asked. If the response has `truncated: true`, use omitted counts to narrow the query or fetch a specific digest by id (call `load_context_digest` with `id`). Treat returned `working_set` paths as candidates for follow-up retrieval, not as automatically trusted or automatically loaded source. Do not call these tools proactively unless the user explicitly enables that behavior.
