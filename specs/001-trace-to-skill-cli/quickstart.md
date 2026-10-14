# Quickstart Validation Guide

This guide defines end-to-end validation scenarios for the planned CLI. It is not an installation
guide for a released package.

## Prerequisites

- Node.js 22 or later and npm.
- A configured OpenAI-compatible model endpoint, model name, and API key.
- A temporary fixture trace directory; tests MUST NOT operate on the user's real session store.

The current planning environment reports Node.js 20.16. Upgrade to a supported Node LTS before
running the project after implementation.

## Build and test

From the repository root:

```sh
npm install
npm test
npm run typecheck
npm run build
```

Expected: all unit, integration, and CLI contract tests pass; strict type checking passes;
the CLI build completes.

## Validation scenarios

1. **Find a reusable lesson**: Run a scan against fixture traces containing a synthetic
   mistake-and-recovery pattern. Expect one pending proposal with evidence references and
   generalized instructions.
2. **No lesson**: Scan a fixture with no generalizable behavior. Expect a no-lessons message and
   no proposal.
3. **Incremental scan**: Scan an unchanged fixture twice. Expect the second scan not to call the
   model; change one event and expect only that session to be re-analyzed. Repeat with `--all`
   and expect a forced re-scan.
4. **Scope selection**: Provide project-associated and unassociated fixture sessions. Choose
   current-project scope and then all-session scope; expect only associated sessions in the first
   run and both in the second.
5. **Privacy**: Include synthetic API-key-like values, emails, and local paths in fixture
   content. Capture the mocked model request; expect those values to be absent and redaction to
   be reported.
6. **Review gate**: Reject one proposal and interrupt review of another. Expect no skill file;
   the rejection and pending proposal remain in state. Approve a third proposal and expect the
   exact reviewed content at the selected destination.
7. **Conflict protection**: Pre-create a destination file with known content. Expect no change
   unless the user explicitly chooses update-existing.
8. **Failure isolation**: Include malformed JSONL and one missing/unreadable source root.
   Expect actionable diagnostics while valid traces from other roots remain analyzable.
9. **Malformed model response**: Return an invalid response from the mock endpoint. Expect an
   explicit error and no successful proposal fallback.

Detailed entity and validation rules are in [data-model.md](./data-model.md); public command
behavior is in [contracts/cli.md](./contracts/cli.md), and source record requirements are in
[contracts/trace-jsonl.md](./contracts/trace-jsonl.md).
