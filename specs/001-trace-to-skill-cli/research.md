# Research: Trace-to-Skill Learning CLI

**Date**: 2026-10-14

## Runtime and project setup

**Decision**: Use TypeScript strict mode and target Node.js 22 or later. Ship against supported
Node LTS versions and declare the minimum runtime in package metadata.

**Rationale**: Node.js 20 is end-of-life as of the planning date. The current local environment
reports Node 20.16, so local implementation validation requires a supported Node upgrade.
Targeting active LTS reduces security and maintenance risk.

**Alternatives considered**: Retain Node 20 compatibility for the current machine; rejected
because the runtime is out of support.

## CLI and test tools

**Decision**: Use Commander for a small, explicit command set and built-in `node:test` with
temporary-directory integration tests and child-process CLI tests.

**Rationale**: The tool needs a few stable commands, not a plugin ecosystem. The built-in test
runner avoids adding a test framework dependency; black-box tests can validate output, exit
status, state, and skill-file side effects through the public CLI.

**Alternatives considered**: Oclif offers plugin/scaffolding features that are not currently
required. Vitest offers richer snapshots and developer experience but adds a dependency; revisit
if snapshots materially improve skill-output tests.

## Boundary validation and normalized trace records

**Decision**: Define Zod schemas for command/config inputs, session metadata, each supported
trace-record format, persisted state, model responses, and skill frontmatter. Convert validated
input into one internal normalized trace representation.

**Rationale**: Source formats are untrusted and vary by agent. Parsing at each boundary satisfies
the constitution while a normalized internal representation keeps learning logic independent of
source formats.

**Alternatives considered**: Pass agent-specific raw records into the learning logic; rejected
because it spreads format coupling and makes consistent validation harder.

## Trace discovery and streaming

**Decision**: Discover the default Copilot session-state directory plus user-configured roots.
Implement a Copilot directory adapter for session folders containing metadata and event streams,
and a documented generic JSONL adapter. Stream event lines and validate records independently.
Compute content fingerprints to identify changed sessions without sending unchanged sessions to
the model.

**Rationale**: Local inspection confirmed `~/.copilot/session-state` uses per-session directories
with `workspace.yaml` and, for some sessions, `events.jsonl`. The actual event contents were not
read for planning. Streaming bounds memory by record size; per-record validation allows one bad
line to be reported without discarding a whole session. Content fingerprints are more reliable
than modification times alone for detecting same-size edits.

**Alternatives considered**: Load entire files in memory; rejected because traces can be large.
Use only an opaque timestamp or file-size cursor; rejected because changes can preserve those
metadata values.

## State storage and crash safety

**Decision**: Use a versioned JSON state document under an OS-appropriate user data directory,
written atomically and protected during read-modify-write with a local lock. Include a
`schemaVersion` and explicit sequential migrations. Keep API keys out of persisted state.

**Rationale**: The feature is a single-user local CLI and its state is modest enough for a
human-readable document. Atomic replacement and locking prevent partial writes and concurrent
scan/review races; versioning permits safe future changes.

**Alternatives considered**: SQLite offers transactions and queryable history but adds native
dependency/distribution and schema-management complexity not justified by the current single-user
scope. A plain JSON overwrite without locking or atomic replacement risks corrupting state.

## Model-service boundary

**Decision**: Keep model access behind a provider interface. The initial adapter uses an
OpenAI-compatible chat-completions HTTP contract with endpoint, model name, and credential
provided through environment variables. Validate the response with Zod and reject malformed or
incomplete output.

**Rationale**: A provider adapter keeps the core flow independent of any one service while
allowing implementation and contract tests to use a deterministic local mock endpoint.
Configuration is not committed or copied into trace/state files.

**Alternatives considered**: Bind the CLI directly to one provider SDK; rejected because no
provider was specified and this would reduce portability. Add a general plugin system now;
rejected as unnecessary for the initial adapter boundary.

## Privacy screening

**Decision**: Apply deterministic secret and personal-data pattern screening before constructing
any model request; redact matches, generalize local paths/project identifiers, and warn when
redactions occur. The CLI clearly discloses that remaining screened session content is sent to
the configured model endpoint.

**Rationale**: The spec permits screened content to go to the user-configured service but
requires sensitive values not be sent or included in a skill. Deterministic transformations are
repeatable and directly testable. The human review gate provides a final check before installation.

**Alternatives considered**: Rely solely on the model to omit sensitive content; rejected because
raw data would already have left the machine. Never send any traces externally; rejected because
the clarified feature uses a configured AI service.

**Known limit**: Pattern detection cannot prove that every possible secret or personal datum has
been found. Keep the scanner modular, report detected categories, and do not claim guarantees
beyond tested patterns.

## Skill writes and proposal lifecycle

**Decision**: Persist proposals and their statuses locally. Show exact content and destination
before approval, edit through the user's configured editor or an equivalent interactive editor,
and write an approved skill through a same-directory temporary file followed by atomic rename.
Never replace an existing path unless the user explicitly selects the update action.

**Rationale**: Approval must be explicit, survive process interruption, and make no unapproved
changes. Atomic writes reduce partial skill files; conflict handling preserves existing content.

**Alternatives considered**: Auto-install generated skills; rejected by FR-006 and the review
requirement. Replace matching skill files silently; rejected as destructive and unsafe.

## Sources

- Node.js release schedule: https://nodejs.org/en/about/previous-releases
- Node.js `readline`: https://nodejs.org/api/readline.html
- Node.js test runner: https://nodejs.org/api/test.html
- Zod basics: https://zod.dev/basics
- Commander documentation: https://commander.js.org/
- JSON Lines format: https://jsonlines.org/
- `write-file-atomic` project: https://github.com/npm/write-file-atomic
- `proper-lockfile` package: https://www.npmjs.com/package/proper-lockfile

