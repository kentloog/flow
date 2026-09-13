# Reviewer access and continuation

Read this during setup, execution preflight or review. The workflow contract is the same in Claude Code Desktop, Claude Code CLI, Codex desktop and Codex CLI. These instructions select capabilities; they do not require a particular tool name, model version or background supervisor.

## Select the reviewer

| `review.reviewer` | Selection |
|-------------------|-----------|
| `auto` (default) | Claude for Codex-led implementation; Codex for Claude-led implementation |
| `claude` / `codex` | That provider, through an available bridge or CLI |
| `current` | Explicit self-review; report the independence limitation |
| A command | The configured reviewer; verify its actual provider and continuation interface |

Use known implementation provenance, not the app's name, to select the opposite family. A Claude subagent using another Claude model is still the same family. For standalone review with unknown authorship, select the opposite of the current coordinator and disclose the limit. If neither gives a Claude/Codex family, resolve reviewer choice before AFK execution. For mixed authorship, use an independent session and report who implemented and validated each area without claiming the reviewer family authored none of it.

Honor existing authorization for repository review, including an explicit request to use both providers. Resolve missing credentials or provider authorization during setup or planning. If a required reviewer cannot run, record the blocker; implementation can continue where useful, but review and readiness remain incomplete. An explicit `current` choice can relax the default, unless the user has required cross-model review.

## Prefer an available review bridge

Discover callable tools and read their schemas. In Codex, a Claude Reviewer bridge may expose `claude` and `claude_reply`; in Claude Code, a Codex MCP bridge may expose `codex` and `codex-reply`. These are examples, not guaranteed tool names. Use the bridge's declared fields for starting and continuing a session. Follow its permission and model-selection rules; leave model choice to native configuration unless the user requested an override.

Give the reviewer read-only scope and the self-contained brief from `./review-steps.md`. It must not edit, commit, update shared state or invoke another reviewer. Prefer a bridge restricted to review tools; prevent recursive Claude-to-Codex-to-Claude delegation. Only the coordinator starts follow-up calls and applies fixes.

Record provider, transport, working directory and session ID in `execution.reviewer` as soon as available, separately from transient `execution.active` entries. Record a running call's native/process handle in `active` immediately when returned. Keep prompts and outputs in `logs/` with distinct pass filenames; reports contain conclusions, not full transcripts.

## CLI fallback

If the bridge is absent or fails, use the selected provider's installed authenticated CLI from the same local workspace when available and authorized. A desktop session can invoke that CLI through its shell tool; it does not need to become a CLI session. Remote sessions need reviewer access on their execution host. Inspect installed help before invoking it: flags and output formats can change.

Write the brief to an absolute-path file and pass it through stdin. For a reviewer without shell access, the coordinator saves the diff to a file and supplies its path plus source paths. Preserve the user's model configuration and permission rules. Restrict the review to reads; do not use bypass flags or grant broader permissions to solve a failed invocation.

Examples below assume the shell's working directory is the reviewed repo and `review_dir` is an absolute workflow log directory. Use new filenames for each pass. The coordinator creates the prompt/diff files; CLI redirection captures output without allowing the reviewer to write reports.

Claude start and follow-up, with file-reading tools and no nested MCP servers. Generate a UUID as `review_session` and persist it in `execution.reviewer.session_id` before the first call:

```bash
claude -p --session-id "$review_session" --permission-mode plan --tools "Read,Glob,Grep" \
  --strict-mcp-config --mcp-config '{"mcpServers":{}}' --output-format json \
  < "$review_dir/review-1-prompt.md" > "$review_dir/review-1.json"

claude -p --resume "$review_session" --permission-mode plan --tools "Read,Glob,Grep" \
  --strict-mcp-config --mcp-config '{"mcpServers":{}}' --output-format json \
  < "$review_dir/review-2-prompt.md" > "$review_dir/review-2.json"
```

Confirm the returned `session_id` matches the recorded ID and read the final result; inspect reported errors and permission denials. These examples require the coordinator's saved diff because they do not expose Bash. If required inputs cannot be read under the host's permissions, record missing coverage rather than a clean full review.

Before either Codex invocation, inspect the effective MCP/plugin configuration and restrict exposed tools to read-only review. Disable write-capable tools and reviewer delegation through supported per-invocation overrides, including plugin-provided tools, or use a restricted bridge. A filesystem sandbox alone does not satisfy this precondition. Do not assume `-c 'mcp_servers={}'` clears inherited servers; verify the effective configuration. Keep the same restrictions on resume.

After that preflight, Codex start and follow-up use a read-only sandbox with failed permission requests returned to the agent:

```bash
codex -a never -s read-only exec --json \
  -o "$review_dir/review-1-result.md" - \
  < "$review_dir/review-1-prompt.md" > "$review_dir/review-1.jsonl"

codex -a never -s read-only exec resume "$review_session" --json \
  -o "$review_dir/review-2-result.md" - \
  < "$review_dir/review-2-prompt.md" > "$review_dir/review-2.jsonl"
```

Include any resolved tool-restriction overrides in both invocations above. Capture `thread_id` from `thread.started` as the session ID. Check the process exit code and final turn/result; an output file or session ID alone is not success. Never use `--last` or `--continue` for workflow continuation: concurrent sessions make them ambiguous.

## Failure and handoff

A transport timeout is not proof the reviewer stopped. Inspect its recorded handle, session or output before retrying. Wait or recover an active call; do not launch a duplicate or switch transports while liveness is uncertain. Once termination is confirmed, retry a transient failure once, then record a blocker if no authorized route works. Transport recovery does not reset repair budgets.

Session IDs belong to their provider, transport and host. Reuse only a compatible session. When switching apps, bridges or machines, recover completed output first and start a fresh review with the durable brief and latest report if continuation is unavailable. The review loop survives through files even when a native session cannot move between hosts.

Official references: [Claude non-interactive sessions](https://code.claude.com/docs/en/headless), [Claude Desktop configuration](https://code.claude.com/docs/en/desktop), and [Codex non-interactive sessions](https://learn.chatgpt.com/docs/non-interactive-mode). The examples were checked against installed command help on 2026-09-13; use current help at runtime.
