# MoContext AI Session Starter

Use MoContext during this session when local retrieval or continuity would help.

MoContext exposes the same surface through MCP tools/resources and a local REST API. Prefer MCP when tools are visible in the AI client; otherwise use REST at `http://127.0.0.1:43210/v1`. All request/response fields are `snake_case`.

At session start, one call orients you: `get_server_summary` (REST `GET /server-summary`) returns status, budgets/capabilities, and the sys-doc/memory folder paths. Call `get_usage_guide` or read `mocontext://docs/ai-usage` for per-tool guidance and workflows. Call `get_activity_briefing` when prior-work context would help.

Trigger words — act on these only when the user types them or has explicitly enabled automatic long-term memory:

- `LoadContext`: call `load_context_digest` with no arguments (REST `GET /context/digest`).
- `LoadContext <keyword>`: call `load_context_digest` with `q=<keyword>`; pass `id=<match id>` afterwards for a full record.
- `SaveContext`: call `save_context_digest` (REST `POST /context/digest`) with the schema-v2 shape — dense `major_work` (concrete changes + touched files), `decisions_v2` (decision + rationale), typed `findings`, `pending_work`. The server rejects digests that are too thin (summary under 200 chars with no major_work and no decisions); if rejected, expand the digest and retry — it is not a transient error. See the usage guide for the full JSON shape.

A good habit for the human: type `SaveContext` after a commit, at a milestone, or before ending a session, so the next session can `LoadContext` its way back in.

During work: use `find_files` to discover files, `get_evidence_pack` for grounded snippets before factual claims (pass `working_dir` to scope to the active project), and `read_file` when snippets are not enough. Treat digests, activity notes, memory files, and `working_set` paths as orientation, not proof — re-read actual source before code changes or conclusions.

At milestones: `record_activity` with a short factual `summary_text` and one `next_step_text` — only at meaningful milestones, not after every tool call. Use `write_memory_file` only for durable, human-editable `.md`/`.txt` notes under the user-owned `memory` folder. Never write into `sys`; it is product-owned reference material.

After a context compaction: if you notice your context was compacted (a continuation summary replaces earlier turns), call `record_activity` once before resuming — concrete summary of completed work (files, commits, decisions), the exact next step, and a `topic_key`. Do not auto-save a digest; digests stay behind the `SaveContext` trigger word.

Do not assume hardcoded install or AppData paths — `get_server_summary` returns the exact local folders. If MoContext is unavailable, say so clearly and continue with normal file reads and code inspection.
