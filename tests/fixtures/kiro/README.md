# Kiro 2.27.1 captures

These fixtures are reduced, sanitized captures from local Kiro CLI 2.27.1
stores, read on 2026-10-06. The original top-level fixtures came from existing
history; the `v3/` fixtures came from an explicit `--agent-engine v3` smoke test
using a disposable HOME and database copy. Source stores were opened read-only.

- `cli.json` / `cli.jsonl`: one complete user turn in the flat CLI format from
  `~/.kiro/sessions/cli/<uuid>.json{,l}`. Metadata retains the v1 marker,
  subagent origin, model, and turn usage/message-id association. JSONL retains
  prompts, thinking, assistant text, toolUse, and toolResult blocks in order.
  The redundant `ToolResults.data.results` transport map was removed.
- `session.json` / `messages.jsonl`: a workspace session from
  `~/.kiro/sessions/<workspace-hash>/sess_<id>/`. The captured schema is
  `1.0.0` (dataModelVersion 1). It contains a prompt and a Say response,
  session notes, and credit usage. No tool event occurs in this capture.
- `conversation.json`: one `conversations_v2` row from the macOS platform
  store. `value` is represented as JSON here; tests serialize it into the exact
  TEXT column of a temporary SQLite database. The conversation includes tools
  and results. Context, environment, and tool registry fields were removed.
- `v3/session.json`, `v3/messages.jsonl`, `v3/sub-executions/*.jsonl`: native V3
  file reads, shell execution, a one-stage `orchestrate_subagent`, child messages,
  Say/Reasoning operations, and credit usage. Tool-call and execution identities
  remain consistent across the parent and child captures. The child start event
  precedes the parent's orchestration tool call, as it does in native output.
- `v3/compaction.jsonl`: the native assistant `Summary` event produced by the
  TUI `/compact` command. Context-only tombstones are not session deletion evidence.
- `v3/tangent/`: a separate native tangent session created with `/tangent smoke`,
  retaining `parentSessionId`, `forkedAtMessageId`, and `createdReason: tangent`.

All text, titles, paths, command arguments, signatures, timestamps, and ids
were replaced. Shared native ids remain shared after replacement. Numeric
usage and enum discriminants retain their original shape. Null classic usage
and zero CLI usage reflect what this version saved, not invented token totals.
Tests mutate copies of these captures for explicit regression cases.

The `.history` companions use the `#V2` line-editor history format. They repeat
prompt input and do not contain a complete transcript, so they are not indexed.

Ground truth: [Kiro session management](https://kiro.dev/docs/cli/chat/session-management/)
confirms per-directory saving and ids. The public predecessor's
[`GlobalPaths::database_path_static`](https://github.com/aws/amazon-q-developer-cli/blob/main/crates/chat-cli/src/util/paths.rs)
uses `dirs::data_local_dir()/amazon-q/data.sqlite3`. Kiro renamed the
application directory to `kiro-cli`; its [official storage documentation](https://kiro.dev/docs/cli/experimental/knowledge-management/)
lists the corresponding macOS, Linux and Windows local-data directories.
The macOS database path was also verified locally. The newer file layouts and
field shapes above were verified against these local captures; Kiro does not
publish a stable schema contract for them.

The `v1` event marker is a serialization label, not proof of the V1 agent engine.
Separate native engine checks establish the layout mapping for this release:
V1 writes `conversations_v2`, V2 writes flat CLI JSONL, and V3 writes nested
workspace JSONL. Engine numbers are independent of the event/schema labels.
