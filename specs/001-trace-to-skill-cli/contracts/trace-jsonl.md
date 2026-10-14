# Generic Trace JSONL Contract

The generic adapter accepts a directory containing one or more UTF-8 `.jsonl` files. Each
non-empty line is an independent JSON object. One malformed record is reported with file and
line number and does not cause unvalidated content to be analyzed.

## Required common record fields

```json
{
  "schemaVersion": 1,
  "sessionId": "stable-session-id",
  "timestamp": "2026-10-14T09:00:00.000Z",
  "type": "user_message",
  "content": "A user request or agent action"
}
```

- `schemaVersion`: positive integer supported by the selected adapter.
- `sessionId`: non-empty identifier consistent within a session.
- `timestamp`: ISO-8601 timestamp.
- `type`: one of `user_message`, `assistant_message`, `tool_call`, `tool_result`, or `metadata`.
- `content`: string; may be empty for metadata records, but not for conversational records.
- Optional `projectRoot`: source project association. If absent, the session is included only
  when the user selects all-session scope.
- Optional `agent`: source-agent label; defaults to `unknown`.
- Optional `toolName` and `toolStatus`: relevant to tool records.

Adapter-specific extensions are permitted only when declared by an adapter schema. Unknown
formats are reported as unsupported rather than guessed.

The reader streams lines, applies size limits per record, validates each object with Zod, and
retains only redacted evidence for generated proposals.

