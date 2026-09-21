# MoContext release history

MoContext ships inside the Mo-Search installer and follows its release cycle, so these version numbers are Mo-Search versions. Only MoContext changes are listed; see the [Mo-Search changelog](https://github.com/Meauxsoft/mo-search/blob/main/CHANGELOG.md) for everything else. Documented through **26.0**.

Generated from the Meauxsoft release history by `build-changelog.ps1`. Do not edit by hand.

## 26.0 — 2026, Sep 21

_A 40-day trial, a $50 license, and MoContext on this PC_

- MoContext: Ships with Mo-Search as the local MCP server. Compatible AI clients search and read from the Mo-Search index on this PC; nothing is uploaded.
- MoContext: Settings backup/restore, daily auto-backup, and a first-run choice to Restore or Start fresh. Default backup folder is Documents\MoBackups.

## 9.50.8 — 2026, Jul 17

_Safer duplicates, dependable indexing, and richer MoContext memory_

- MoContext: Adds structured saved-context sections for major work, decisions and rationale, findings, and pending work.
- MoContext: Generates a readable weekly summary from recent activity and saved context.
- MoContext: Adds a configurable quality check that guides clients when a saved context summary is too thin.
- MoContext: Simplifies the local AI interface to 15 clearer tools while preserving compatibility with existing REST clients.
- MoContext: Combines search, browse, and recent-file discovery into a clearer find_files workflow.
- MoContext: Combines whole-file, line-range, and focused-window reading into a clearer read_file workflow.
- MoContext: Reports when indexed files may have changed since their last index update.
- MoContext: Adds an overall newest-index timestamp so clients can quickly judge search freshness.
- MoContext: Keeps activity from unsessioned clients grouped predictably, with optional client-specific grouping.
- MoContext: Truncates overlong activity text with a clear warning instead of rejecting the entire entry.
- MoContext: Refreshes weekly summaries promptly after later saves without losing background update signals.
- MoContext: Returns clear client errors for zero, negative, or reversed file-reading ranges.
- MoContext: Updates its MCP and SQLite components for improved long-running reliability and current database support.

## 9.50.7 — 2026, Jun 20

_Smoother updates, stronger recovery, and MoContext polish_

- MoContext: Added project-folder scoping to search, browse, and evidence-pack retrieval over HTTP and MCP.
- MoContext: Scoped searches now report which project folder was searched, even when no results are found.
- MoContext: Improved search relevance so small path-only and media records no longer crowd out useful source files.
- MoContext: Redesigned the Status and Explore Context tabs for consistent light and dark themes.
- MoContext: Added normal keyboard navigation plus Alt+S and Alt+E shortcuts for switching tabs.
- MoContext: Added bottom spacing to Explore Context results for cleaner scrolling and presentation.
- MoContext: Updated its SQLite and MCP components to current releases.
- MoContext: Expanded diagnostics to report both managed and native SQLite versions.

## 9.50.6 — 2026, Jun 6

_Search Home, faster queries, MoContext memory, and indexing reliability_

- MoContext: Added long-term context digest save/load/search support backed by human-readable weekly Markdown files.
- MoContext: Added MCP/API tools for `save_context_digest`, `load_context_digest`, and `get_context_digest`.
- MoContext: Added digest startup validation, duplicate-id detection, retention archiving, load budgets, and support-report summaries.
- MoContext: Added UriHistory-based working-set hints so AI clients can see recently active files without loading file contents.
- MoContext: Added compact status UI improvements plus a tabbed Explore Context panel for inspecting LoadContext-style evidence packs.
- MoContext: Expanded support reports with context digest, MCP tool-call, dashboard, and transport diagnostics.
- MoContext: Expanded the API correctness harness for digest, working-set, API, and MCP coverage.

## 9.50.5 — 2026, Apr 26

_MoContext indexed memory, MCP discovery, installer hardening, and FileViewer cleanup_

- MoContext: Added MCP resources/templates and concise front-door calls including server summary, diagnostics, usage guide, context paths, context-file listing/reading, and memory-file writing.
- MoContext: Added per-user AppData context folders with product-owned `sys` docs and user/AI-owned `memory` notes for searchable long-term context.
- MoContext: Installed AI usage docs, human prompt guidance, and the consolidated human guide into both Program Files and the indexed AppData MoContext docs folder.
- MoContext: Split compact health from verbose diagnostics and refreshed dashboard/support-report data for the new diagnostics shape.
- MoContext: Added scoped context-file APIs with safe relative paths, extension limits, max-size checks, and read truncation.
- MoContext: Expanded the installed API smoke test to cover diagnostics, context paths/files, context-file reads, and MCP resource listing/reading.

## 9.50.4 — 2026, Apr 12

_MoContext tools, diagnostics exports, and simple-search polish_

- MoContext: Added MCP tool surface (search, browse, evidence-pack, file-window/full-file, activity/session tools) with event tracking.
- MoContext: HTTP server bootstrap includes MCP transport, OAuth discovery response, and richer transport logging/diagnostics.
- MoContext: Evidence-pack retrieval now runs end-to-end against MoSl.db with ranking, windowing, budgets, and loader gating.
- MoContext: Repository now persists sessions, activity logs, saved packs, and event telemetry.
- MoContext: Support report export bundles health snapshot, dashboard model, and recent transport issue summaries.

---

Full release notes for every version are published at [meauxsoft.com](https://www.meauxsoft.com/MoSearch_Releases.html).
