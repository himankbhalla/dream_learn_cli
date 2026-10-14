# Data Model: Trace-to-Skill Learning CLI

This document describes normalized internal and persisted entities. External data is parsed
from `unknown` with Zod before entering domain logic.

## AgentTrace

A normalized coding-agent session assembled from source metadata and event records.

| Field | Type | Rules |
|---|---|---|
| `id` | string | Required; stable source session identifier within its trace root |
| `sourceId` | string | Required; identifies the configured source adapter/root |
| `sourceAgent` | string | Required; detected agent identifier or `unknown` |
| `formatVersion` | string | Required; adapter-defined format/version |
| `projectRoot` | string or null | Normalized absolute project path when source metadata provides it |
| `startedAt` | ISO-8601 string or null | Must parse as a valid date when present |
| `updatedAt` | ISO-8601 string or null | Must parse as a valid date when present |
| `fingerprint` | hexadecimal SHA-256 string | Hash of the source metadata and event files included in this analysis |
| `records` | ordered `TraceRecord[]` | Validated records; malformed lines are counted/reported, not trusted |

`TraceRecord` is a discriminated union of user request, assistant response, tool activity, and
source metadata records. Adapter-specific fields are mapped to the normalized shape. Records
retain source file and line references for evidence, but raw content is not copied into state.

## TraceSource

Configured local location and adapter used to discover sessions.

| Field | Type | Rules |
|---|---|---|
| `id` | string | Stable generated identifier |
| `rootPath` | absolute path | Required; default includes `~/.copilot/session-state` |
| `adapterId` | string | Required; Copilot adapter or generic JSONL adapter |
| `enabled` | boolean | Defaults to true |

Missing or unreadable roots produce an explicit diagnostic; other configured roots continue to
be processed.

## Lesson

A task-independent insight supported by one or more trace passages.

| Field | Type | Rules |
|---|---|---|
| `statement` | string | Required; generalizable, actionable insight |
| `appliesWhen` | string | Required; describes the reusable situation |
| `evidence` | `EvidenceReference[]` | At least one validated reference to source session/record |
| `confidence` | number | 0 through 1; provider output is validated and displayed, not treated as approval |

## ProposedSkill

A reviewable draft derived from one or more lessons.

| Field | Type | Rules |
|---|---|---|
| `id` | UUID string | Unique proposal identifier |
| `name` | string | Required; safe skill-directory name (lowercase alphanumeric and hyphens) |
| `description` | string | Required; concise applicability summary |
| `instructions` | string | Required; non-empty actionable skill body |
| `evidence` | `EvidenceReference[]` | Required; preserves provenance to source session and record |
| `contentHash` | hexadecimal SHA-256 string | Stable content digest, used to detect duplicate proposals |
| `status` | `pending \| approved \| rejected \| installed` | Only explicit approval advances to installation |
| `createdAt` | ISO-8601 string | Required |
| `updatedAt` | ISO-8601 string | Required |
| `destination` | absolute path or null | Set and shown before approval/install |

Transitions: `pending → approved → installed`, `pending → rejected`, and `pending → pending`
when edited. Update-on-overlap requires an explicit user action and is recorded as an approved
replacement; no transition implicitly overwrites a file.

## EvidenceReference

| Field | Type | Rules |
|---|---|---|
| `traceId` | string | Required; source session identifier |
| `sourceId` | string | Required; identifies the trace root |
| `file` | relative path | Must remain under its configured trace root |
| `record` | positive integer | One-based record/line reference |
| `excerpt` | string | Redacted excerpt only; secrets and local identifying paths excluded |

## ReviewDecision

| Field | Type | Rules |
|---|---|---|
| `proposalId` | UUID string | References one proposal |
| `decision` | `approve \| reject \| edit \| update-existing \| keep-both \| discard` | Explicit user choice |
| `decidedAt` | ISO-8601 string | Required |
| `reason` | string or null | Optional; must not include credentials or unredacted trace data |
| `editedContentHash` | string or null | Required when edited content is approved |

## ScanState

Versioned local persistence envelope.

| Field | Type | Rules |
|---|---|---|
| `schemaVersion` | positive integer | Required; sequential migrations only |
| `sessions` | map of source/session key to session scan record | Stores fingerprint, last completed analysis, and decision references |
| `proposals` | `ProposedSkill[]` | Includes pending proposals so interruption does not lose work |
| `decisions` | `ReviewDecision[]` | Records rejection/approval and prevents repeated suggestions |
| `updatedAt` | ISO-8601 string | Required |

State schema versions are validated individually before migration and current state is validated
again before persistence. Provider secrets and unredacted trace content are never persisted.
