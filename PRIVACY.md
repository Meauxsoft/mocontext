# Privacy

MoContext is a local service distributed with Mo-Search. By default it binds
to `127.0.0.1`, reads the local Mo-Search index, and stores its activity and
memory data on the Windows computer.

MoContext does not upload indexed file content, activity notes, or memory files
to a Meauxsoft cloud service. It does not require a MoContext cloud account.

## The AI-client boundary

When an AI client calls MoContext, the client receives search results, file
paths, snippets, or file contents requested through MoContext's tools. A
cloud-backed client may then transmit that material to its model provider.
That transmission is controlled by the client, not MoContext, and is governed
by the client's and model provider's privacy and retention terms.

A fully local workflow therefore requires both:

1. MoContext running locally.
2. An MCP client and model stack configured to run locally.

Review your AI client's settings and policies before allowing it to retrieve
sensitive material.

## Data under your control

Mo-Search indexes the locations and file types selected in its configuration.
MoContext can search and read content available through that index and its
configured file-access rules. Its human-editable memory files can be inspected,
edited, backed up, or deleted by the user.

Do not publish diagnostics, screenshots, logs, search results, file paths, or
memory files without reviewing them for private information.

Questions may be sent to
[Questions@meauxsoft.com](mailto:Questions@meauxsoft.com).
