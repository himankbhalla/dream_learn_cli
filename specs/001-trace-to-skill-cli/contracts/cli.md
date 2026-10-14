# CLI Contract

This contract is the user-facing command shape for the initial release. Exact command spelling
may be finalized during task breakdown, but behaviors and safety guarantees are normative.

## Commands

| Command | Behavior |
|---|---|
| `trace-skill scan` | Discover configured roots, prompt for current-project or all-session scope, and analyze only new/changed sessions. |
| `trace-skill scan --all` | Force re-analysis of sessions in the selected scope. |
| `trace-skill review [proposal-id]` | List pending proposals or review the selected proposal. Show full skill content and destination; support approve, reject, edit, update existing, keep both, or discard as applicable. |
| `trace-skill config sources` | List or configure additional local trace roots and their adapter format. |
| `trace-skill config skills-dir [path]` | Show or set the default skill destination; defaults to the current project's `.github/skills`. |
| `trace-skill status` | Report pending proposals, last scan, enabled roots, and skipped/malformed trace counts without printing secrets or raw trace content. |
| `trace-skill --help` | Display available commands and safe usage examples. |

## Exit and output behavior

- Successful scan/review exits `0`, including a successful scan that finds no useful lesson.
- Invalid command/configuration or unavailable required model configuration exits non-zero with
  an actionable message on stderr.
- Per-root or per-record failures are reported; a failure in one source does not silently hide
  successful results from other readable sources.
- Non-interactive execution may create pending proposals but MUST NOT approve or install them.
- No command prints API keys, secrets, or unredacted trace excerpts.

## Approval and installation

Before approval, display the complete proposed content and exact destination. Approval is a
separate explicit action. Existing files are never replaced unless the user selects an explicit
update-existing action. Interrupting review leaves the proposal pending and installs nothing.

