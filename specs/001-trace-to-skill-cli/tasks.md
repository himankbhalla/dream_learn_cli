# Tasks: Trace-to-Skill Learning CLI

**Input**: Design documents from `specs/001-trace-to-skill-cli/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Included because the project constitution requires test-first development.

**Organization**: Tasks are grouped by user story so each deliverable can be implemented and
validated incrementally.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel with other ready tasks because it changes different files and has
  no dependency on their incomplete work.
- **[Story]**: Required for user-story tasks; setup, foundational, and polish tasks omit it.
- Every task names the exact file or files it creates or changes.

## Phase 1: Setup

**Purpose**: Initialize the strict TypeScript CLI project and repeatable build/test commands.

- [ ] T001 Create the Node.js CLI package manifest with Node.js `>=22`, Commander, Zod, `write-file-atomic`, `proper-lockfile`, TypeScript, and `@types/node` in `package.json`.
- [ ] T002 Configure strict TypeScript compilation, ES module output, Node.js 22 target, and `src`/`dist` paths in `tsconfig.json`.
- [ ] T003 Add `build`, `typecheck`, `test`, and `start` scripts plus the executable `trace-skill` entry point in `package.json`.
- [ ] T004 [P] Create the source and test directory layout from `plan.md` with a minimal `src/cli/index.ts` entry point and `.gitignore` entries for `node_modules/`, `dist/`, and local test artifacts.

---

## Phase 2: Foundational

**Purpose**: Implement cross-story types, validated config, error reporting, and safe local state.

**⚠️ CRITICAL**: Complete this phase before user-story work.

- [ ] T005 [P] Write and run failing unit tests for source configuration, provider configuration, ISO timestamps, fingerprints, and state schema-version validation in `tests/unit/config-schema.test.ts` and `tests/unit/state-schema.test.ts`.
- [ ] T006 [P] Define Zod schemas and inferred TypeScript types for `AgentTrace`, `TraceRecord`, `TraceSource`, `Lesson`, `ProposedSkill`, `EvidenceReference`, `ReviewDecision`, and `ScanState` in `src/shared/schemas.ts`.
- [ ] T007 Implement Zod-validated environment and user configuration loading, including defaults for `~/.copilot/session-state` and this project's `.github/skills`, in `src/config/config.ts`.
- [ ] T008 Implement typed user-facing errors and diagnostics for invalid input, unavailable sources, malformed records, provider failures, and write conflicts in `src/shared/errors.ts` and `src/shared/diagnostics.ts`.
- [ ] T009 Write and run failing state-repository tests for initial state, proposal/session lookups, serialized updates, and corrupted-state errors in `tests/unit/state-repository.test.ts`.
- [ ] T010 Implement OS-appropriate local state paths, versioned state loading, sequential migration hooks, atomic writes, and lock-protected read-modify-write operations in `src/state/state-store.ts` and `src/state/migrations.ts`.
- [ ] T011 Add safe path normalization and containment checks for configured trace roots and skill destinations in `src/shared/paths.ts` and `tests/unit/paths.test.ts`.

**Checkpoint**: Strict project setup, shared types, config, error reporting, and durable state are ready.

---

## Phase 3: User Story 1 - Propose a skill from an agent trace (Priority: P1) 🎯 MVP

**Goal**: Read new or changed sessions, identify generalizable lessons, and persist reviewable
skill proposals without installing them.

**Independent Test**: Scan a temporary Copilot session fixture with an error-and-recovery pattern;
verify a pending proposal with generalized instructions and evidence references is persisted.
Verify a no-lesson fixture creates no proposal and a second unchanged scan does not call the model.

### Tests for User Story 1

- [ ] T012 [P] [US1] Write and run failing Copilot session-folder discovery and metadata parsing tests using synthetic `workspace.yaml` fixtures in `tests/unit/copilot-adapter.test.ts`.
- [ ] T013 [P] [US1] Write and run failing generic JSONL record parsing tests for all declared record types, unknown formats, malformed lines, and record limits in `tests/unit/jsonl-adapter.test.ts`.
- [ ] T014 [P] [US1] Write and run failing privacy tests for API-key patterns, emails, personal data, local paths, and redacted evidence in `tests/unit/redaction.test.ts`.
- [ ] T015 [P] [US1] Write and run failing model-provider contract tests for valid lesson responses, empty lessons, invalid JSON, invalid schemas, unsupported evidence references, HTTP errors, and timeouts in `tests/integration/model-provider.test.ts`.
- [ ] T016 [P] [US1] Write and run failing incremental-scan and proposal-generation tests for changed fingerprints, unchanged sessions, `--all`, zero lessons, and independent evidence in `tests/integration/scan-service.test.ts`.
- [ ] T017 [US1] Write and run a failing black-box `scan` CLI test asserting pending proposal output and no skill installation in `tests/cli/scan.test.ts`.

### Implementation for User Story 1

- [ ] T018 [P] [US1] Implement streaming Copilot session discovery and metadata/event normalization from per-session directories in `src/traces/adapters/copilot.ts`.
- [ ] T019 [P] [US1] Implement the generic UTF-8 JSONL adapter for `schemaVersion`, `sessionId`, `timestamp`, `type`, and `content` records in `src/traces/adapters/jsonl.ts`.
- [ ] T020 [US1] Implement source adapter registration, root enumeration, and failure-isolated discovery across configured trace roots in `src/traces/discovery.ts` and `src/traces/registry.ts`.
- [ ] T021 [US1] Implement deterministic session fingerprints, current-project/all-session filtering, unchanged-session skipping, and forced re-scan handling in `src/traces/scan-cursor.ts`.
- [ ] T022 [US1] Implement sensitive-data detection, redaction, project-path generalization, and redacted evidence extraction before any outbound request in `src/privacy/redact.ts`.
- [ ] T023 [US1] Implement the OpenAI-compatible HTTP provider adapter with environment-configured endpoint, model, and credential plus Zod response validation in `src/learning/providers/openai-compatible.ts`.
- [ ] T024 [US1] Implement the lesson extraction prompt and validate that response evidence refers only to submitted trace records in `src/learning/lesson-extractor.ts`.
- [ ] T025 [US1] Implement normalized trace-to-proposal conversion with safe lowercase hyphenated names, non-empty instructions, SHA-256 content hashes, evidence references, and `pending` status in `src/proposals/proposal-builder.ts`.
- [ ] T026 [US1] Implement the scan orchestration to disclose outbound model use, report redactions and malformed records, save fingerprints only after completed processing, and persist proposals in `src/traces/scan-service.ts`.
- [ ] T027 [US1] Implement `trace-skill scan`, interactive scope selection, `--all`, no-lessons output, diagnostics, and non-interactive pending-only behavior in `src/cli/commands/scan.ts` and `src/cli/index.ts`.

**Checkpoint**: The primary scan flow creates durable pending proposals, keeps unchanged sessions
from being reprocessed, and installs no skills.

---

## Phase 4: User Story 2 - Human review and approval gate (Priority: P1)

**Goal**: Let the developer inspect, edit, approve, or reject proposals; install only explicitly
approved content without overwriting existing skills silently.

**Independent Test**: Review a seeded pending proposal; reject one and confirm no file appears,
interrupt another and confirm it stays pending, then approve one and verify exact approved
content at the displayed destination.

### Tests for User Story 2

- [ ] T028 [P] [US2] Write and run failing proposal lifecycle tests for pending, edited, approved, rejected, installed, and interrupted states in `tests/unit/proposal-lifecycle.test.ts`.
- [ ] T029 [P] [US2] Write and run failing skill writer tests for exact content, safe skill directory names, existing-file conflict, explicit update, unwritable destinations, and atomic-write failure in `tests/unit/skill-writer.test.ts`.
- [ ] T030 [US2] Write and run failing black-box review tests verifying complete content/destination display and that only explicit approval writes a skill in `tests/cli/review.test.ts`.

### Implementation for User Story 2

- [ ] T031 [P] [US2] Implement proposal status transitions and persisted review decisions, including rejection deduplication and pending recovery after interruption, in `src/proposals/proposal-repository.ts`.
- [ ] T032 [P] [US2] Implement skill frontmatter/body rendering and Zod validation for installed skill content in `src/skills/skill-format.ts`.
- [ ] T033 [US2] Implement atomic skill installation, destination validation, and conflict detection that never replaces an existing file without explicit update-existing approval in `src/skills/skill-writer.ts`.
- [ ] T034 [US2] Implement interactive proposal listing and review choices for approve, reject, edit, update-existing, keep-both, and discard in `src/proposals/review-service.ts`.
- [ ] T035 [US2] Implement `trace-skill review [proposal-id]` with exact pre-approval content/destination display and non-interactive no-install behavior in `src/cli/commands/review.ts`.

**Checkpoint**: A human can safely review and install a skill; rejection and interruption never write files.

---

## Phase 5: User Story 3 - Work with traces from any coding agent (Priority: P2)

**Goal**: Configure additional local agent trace roots and install skills into compatible,
user-selected skill directories.

**Independent Test**: Configure a second generic JSONL root, run a scope including it, and verify
its records produce a proposal; select an alternate skill destination and verify approved
content is written there in the common skill layout.

### Tests for User Story 3

- [ ] T036 [P] [US3] Write and run failing tests for adding, listing, disabling, and validating additional trace roots and adapter IDs in `tests/unit/source-config.test.ts`.
- [ ] T037 [P] [US3] Write and run failing tests for configured skill destinations, destination-specific layout selection, and missing/unwritable target diagnostics in `tests/unit/skill-destination.test.ts`.
- [ ] T038 [US3] Write and run failing CLI tests for source configuration and alternate skill-directory configuration in `tests/cli/config.test.ts`.

### Implementation for User Story 3

- [ ] T039 [P] [US3] Implement versioned, Zod-validated persistence for configured source roots and adapter IDs in `src/config/source-config.ts`.
- [ ] T040 [P] [US3] Implement skill destination adapters for the common `SKILL.md` layout and compatible selected agent directories in `src/skills/destination-adapters.ts`.
- [ ] T041 [US3] Implement `trace-skill config sources` and `trace-skill config skills-dir [path]` commands with explicit path validation in `src/cli/commands/config.ts`.
- [ ] T042 [US3] Connect configured adapters and destination selection to scan/review services without changing their normalized trace and proposal interfaces in `src/cli/index.ts`.

**Checkpoint**: Multiple local agent trace directories can feed the same flow, and approved skills can be sent to a selected compatible destination.

---

## Phase 6: User Story 4 - Avoid duplicate or conflicting skills (Priority: P3)

**Goal**: Detect overlapping installed skills and let the developer decide whether to update,
keep both, or discard a proposal.

**Independent Test**: Place an existing skill that covers a pending proposal, review it, and verify
the overlap is shown and no content changes until the user explicitly selects an action.

### Tests for User Story 4

- [ ] T043 [P] [US4] Write and run failing overlap-detection tests for exact content hashes, normalized names, and semantically similar skill summaries in `tests/unit/skill-overlap.test.ts`.
- [ ] T044 [US4] Write and run failing review integration tests for update-existing, keep-both, and discard outcomes without unintended file changes in `tests/integration/overlap-review.test.ts`.

### Implementation for User Story 4

- [ ] T045 [P] [US4] Implement installed-skill inventory and deterministic exact/name overlap detection in `src/skills/skill-index.ts`.
- [ ] T046 [US4] Implement proposal-to-existing-skill overlap presentation and update/keep-both/discard decision handling in `src/skills/overlap-service.ts` and `src/proposals/review-service.ts`.

**Checkpoint**: Potential duplicates are surfaced and resolved only through an explicit review decision.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Complete operational visibility, documentation, and end-to-end validation.

- [ ] T047 [P] Implement `trace-skill status` with pending proposal counts, last scan, enabled roots, and malformed/skipped record counts while excluding raw traces and secrets in `src/cli/commands/status.ts`.
- [ ] T048 [P] Add end-to-end fixtures and documentation for the generic JSONL format, model configuration, local privacy behavior, review safety, and alternate roots in `README.md` and `tests/fixtures/`.
- [ ] T049 Add safe source traversal limits, symlink/path-escape rejection, bounded JSONL record sizes, and redacted diagnostics in `src/traces/discovery.ts`, `src/traces/adapters/jsonl.ts`, and `src/shared/paths.ts`.
- [ ] T050 Run all scenarios in `specs/001-trace-to-skill-cli/quickstart.md`, fix failures, and record the final supported Node.js runtime and results in `specs/001-trace-to-skill-cli/quickstart.md`.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; must complete before foundational work.
- **Foundational (Phase 2)**: Depends on Setup and blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational; delivers the MVP scan-to-proposal flow.
- **User Story 2 (Phase 4)**: Depends on Foundational and consumes proposals produced by US1;
  can be independently tested using seeded proposals.
- **User Story 3 (Phase 5)**: Depends on Foundational and reuses US1/US2 adapter interfaces;
  full end-to-end verification follows US1 and US2.
- **User Story 4 (Phase 6)**: Depends on US2 skill installation and review lifecycle.
- **Polish (Phase 7)**: Depends on the stories selected for the release.

### User Story Dependencies

- **US1 (P1)**: First; no dependency on another user story.
- **US2 (P1)**: Functionally follows proposals from US1; independently testable with seeded pending proposals.
- **US3 (P2)**: Depends on shared source/destination boundaries; integrates with US1 and US2.
- **US4 (P3)**: Depends on US2's installed-skill inventory and review actions.

### Parallel Opportunities

- After T001-T003, T004 can proceed independently.
- In the foundational phase, T005, T006, T008, and T011 are separable initial file groups; state work T009-T010 follows its test/schema prerequisites.
- For US1, adapter fixtures/tests T012-T014 and provider/scan tests T015-T017 can be developed in parallel; T018 and T019 are parallel adapters; privacy/provider/proposal modules can proceed in parallel after their tests and shared schemas.
- For US2, T028 and T029 are independent tests; format and lifecycle work can proceed separately, then converge at review integration.
- For US3, source-root configuration (T036/T039) and skill destination configuration (T037/T040) are independent; CLI wiring follows both.
- US1 and seeded-proposal US2 work can proceed in parallel after Foundational if ownership is split; US3 can begin adapter/config work after Foundational. US4 follows US2.
- T047 and T048 are independent polish tasks; final quickstart validation T050 follows all selected implementation work.

### Dependency Graph

```text
Setup → Foundational → US1 → US2 → US4 → Polish
                          └→ US3 ─────────→ Polish
```

## Parallel Execution Examples

### User Story 1

```text
Parallel: T012, T013, T014, T015, T016
Parallel after shared schemas: T018, T019, T022, T023
Then integrate: T020 → T021 → T024 → T025 → T026 → T027
```

### User Story 2

```text
Parallel: T028, T029
Then implement: T031, T032 → T033
Integrate review flow: T034 → T035
```

### User Story 3

```text
Parallel: T036, T037
Then implement: T039, T040
Integrate: T041 → T042
```

### User Story 4

```text
Start T043 in parallel with T044 fixtures/design; complete T043/T044 before T045/T046.
```

## Implementation Strategy

### MVP First (User Story 1)

1. Complete Setup and Foundational phases.
2. Implement the Copilot adapter, generic JSONL adapter, privacy screening, configured model
   adapter, and pending-proposal persistence.
3. Validate scan scope, unchanged-session skipping, forced scans, malformed records, redaction,
   zero-lesson behavior, and proposal evidence using temporary fixtures.
4. **Stop and validate US1 independently** before adding approval/install behavior.

### Incremental Delivery

1. Deliver US1: session traces become pending skill proposals.
2. Deliver US2: human review and approval-gated skill installation.
3. Deliver US3: configured additional agent sources and destinations.
4. Deliver US4: overlap detection and explicit resolution.
5. Complete cross-cutting status, documentation, safety, and quickstart validation.
