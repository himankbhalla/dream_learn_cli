# Model Service Contract

The initial provider adapter uses an OpenAI-compatible chat-completions JSON contract. The
endpoint, model, and API key are supplied by the user through environment configuration and are
never persisted in scan state. No trace content is sent until the CLI has disclosed the outbound
data flow and completed deterministic screening/redaction.

## Request

Send only the normalized, redacted trace representation needed to derive reusable lessons.
Requests ask for zero or more lessons, each including a generalizable statement, applicability,
actionable skill instructions, and supporting evidence references. Project paths, identifiers,
and detected secrets are replaced or omitted before the request is constructed.

## Response

The provider response MUST be parsed as untrusted data and validated with Zod. The accepted
payload is a JSON object with a `lessons` array; each lesson includes:

- `statement`: non-empty, task-independent lesson.
- `appliesWhen`: non-empty applicability description.
- `skillName`: a safe lowercase, hyphenated name.
- `description`: non-empty skill description.
- `instructions`: non-empty instruction body.
- `evidence`: one or more references to record identifiers from the submitted input only.
- `confidence`: numeric value from 0 to 1.

Reject references not present in the submitted trace and reject malformed responses. A valid
empty `lessons` array means no generalizable lesson was found. Provider/network/timeout errors
are surfaced without converting them into empty successful results.

