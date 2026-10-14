# Implementation Plan: Trace-to-Skill Learning CLI

**Branch**: `001-trace-to-skill-cli` | **Date**: 2026-10-14 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-trace-to-skill-cli/spec.md`; user selected Node.js with TypeScript for the CLI.

## Summary

Build a cross-platform Node.js CLI that discovers Copilot session folders under
`~/.copilot/session-state` and configured local trace roots, normalizes supported agent
records, screens and redacts sensitive content, and submits only eligible new or changed
sessions to a configured model endpoint. Persist scan cursors and proposals in a versioned,
atomically written local state file. Keep every proposal pending until an interactive user
reviews it; install approved skills into `.github/skills` by default, with explicit support
for alternate agent skill directories.

The implementation starts with a Copilot session-folder adapter and a documented generic
JSONL adapter. Provider and agent adapters remain behind narrow interfaces so support can
grow without coupling the core learning and review flows to a single trace format or model.

## Technical Context

**Language/Version**: TypeScript, strict mode; Node.js 22 or later active LTS

**Primary Dependencies**: Zod for boundary schemas; Commander for CLI commands;
`write-file-atomic` and `proper-lockfile` for safe local persistence; built-in
`node:readline/promises`, `node:test`, and TypeScript compiler for terminal interaction and
test/build

**Storage**: Versioned JSON state in the OS user data directory; skill files in the selected
skills directory. State includes scan fingerprints, pending proposals, and review decisions.

**Testing**: `node:test` with isolated temporary directories; black-box CLI tests using
`node:child_process`; strict `tsc --noEmit` and compiled CLI smoke tests

**Target Platform**: macOS, Linux, and Windows with Node.js 22+; traces and skills remain local
except screened trace content sent to the user-configured model service

**Project Type**: Single TypeScript CLI project

**Performance Goals**: Typical trace-to-review-to-approval flow completes in under five minutes
per the specification; stream trace records so memory use is not proportional to total trace size;
do not re-submit unchanged sessions to the model

**Constraints**: No skill is written before explicit approval; never overwrite without confirmation;
screen and redact sensitive content before external submission; report malformed records and
continue safely where possible; do not persist API secrets in scan state

**Scale/Scope**: One local developer; multiple local trace roots and coding-agent adapters;
initial default source is `~/.copilot/session-state`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Strict TypeScript**: PASS — TypeScript strict mode, no `any`, and type-safe schemas.
- **Test-First**: PASS — tests precede implementation for parsers, state changes, approvals,
  redaction, and writes.
- **Validated API Boundaries**: PASS — Zod validates CLI/config input, trace metadata and
  records, persisted state, model requests/responses, and skill frontmatter before use.
- **Security-First**: PASS — outbound content is screened and redacted; secrets are environment
  configuration only; writes require explicit review/approval and protect existing files.
- **Twelve-Factor Configuration**: PASS — provider endpoint, model, and credential are supplied
  through environment/config; invalid required configuration fails explicitly.
- **Accessible Interfaces**: N/A — no browser UI; prompts and output remain operable and
  understandable in a terminal.
- **Small Modules and Library-First Design**: PASS — trace-source, model-provider, redaction,
  proposal, persistence, and installation responsibilities have focused interfaces.

**Gate result**: PASS. No constitutional exception is needed.

## Project Structure

### Documentation (this feature)

```text
specs/001-trace-to-skill-cli/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   ├── cli.md
│   ├── model-service.md
│   └── trace-jsonl.md
└── tasks.md
```

### Source Code (repository root)

```text
src/
├── cli/
│   ├── index.ts
│   └── commands/
├── config/
├── traces/
│   ├── adapters/
│   ├── discovery.ts
│   └── normalize.ts
├── privacy/
├── learning/
│   └── providers/
├── proposals/
├── skills/
├── state/
└── shared/

tests/
├── fixtures/
├── unit/
├── integration/
└── cli/
```

**Structure Decision**: One CLI project at the repository root. The CLI is a thin command
surface over independently testable trace, privacy, learning, proposal, skill, and state
modules. Provider/agent-specific behavior is implemented as adapters rather than separate
applications.

## Complexity Tracking

No constitution violations or additional projects are proposed.

## Post-Design Constitution Check

- Strict TypeScript and Zod boundary validation are reflected in schemas and normalized entities.
- Test-first implementation and isolated CLI/integration validation are reflected in the quickstart.
- Sensitive-content filtering precedes model I/O; installation is approval-gated and non-overwriting.
- Provider configuration uses environment variables, and each module has one focused boundary.

**Post-design gate result**: PASS. No exception or deferred constitution decision.
