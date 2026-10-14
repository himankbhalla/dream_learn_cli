# Feature Specification: Trace-to-Skill Learning CLI

**Feature Branch**: `001-trace-to-skill-cli`

**Created**: 2026-10-14

**Status**: Draft

**Input**: User description: "I want a cli that takes in the trace of the agents, find a generalizable lesson from it and propose a skill for human review. If approved then add it as a new skill in the skills. It should work with any coding agent. I want to improve my agent with adaptive capabilities using this."

## Clarifications

### Session 2026-10-14

- Q: When the tool reads the session store, should it analyze every stored session on each run, or only sessions it hasn't analyzed before? → A: Analyze only new or updated sessions since the last run; provide an option to force a full re-scan.
- Q: Where should an approved skill be installed by default? → A: This project's `.github/skills` directory by default, while allowing another location to be selected.
- Q: May the tool send selected session traces to the AI service you configure to extract reusable lessons? → A: Send traces to the configured AI service after screening for sensitive data and clearly disclosing this behavior.
- Q: Should the tool analyze sessions for every project recorded in the local session store, or only sessions associated with this project? → A: Read traces from `~/.copilot/session-state` and ask me to choose the session scope on each run: current-project sessions or all sessions.
- Q: To support coding agents beyond Copilot, should the tool also look for their trace files outside `~/.copilot/session-state`? → A: Allow the user to configure additional local trace directories.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Propose a skill from an agent trace (Priority: P1)

A developer runs the tool, which reads available coding-agent session traces from `~/.copilot/session-state` and any additional local trace directories the developer has configured. On each run, the developer chooses whether to analyze sessions associated with the current project or all sessions in the available locations. The tool analyzes only sessions in the chosen scope that are new or changed since the previous run, identifies lessons that would help the agent in future, unrelated tasks (not just the original task), and produces proposed skills: each has a name, a description of when it applies, instructions, and evidence from the trace that motivated it. The developer can request a full re-scan.

**Why this priority**: This is the core value. Without extracting a generalizable lesson and drafting a skill, nothing else matters.

**Independent Test**: Make a new session trace available in `~/.copilot/session-state` containing a recognizable mistake-and-recovery pattern. Choose a scope that includes it and verify a proposed skill is produced that states the lesson in task-independent terms and cites the trace passages that support it.

**Acceptance Scenarios**:

1. **Given** a new or changed session trace where the agent made an error and later corrected it, **When** the user runs the tool, **Then** a proposed skill is shown that describes the lesson generally and references the supporting trace moments.
2. **Given** a trace with no meaningful lesson (e.g., a trivial successful task), **When** the user runs the tool, **Then** the tool reports that no generalizable lesson was found and creates no proposal.
3. **Given** a lesson that is specific to one project's file names or private details, **When** the tool drafts the skill, **Then** the proposed skill omits or generalizes those specifics.

---

### User Story 2 - Human review and approval gate (Priority: P1)

The developer reviews each proposed skill before anything changes. They can approve it, reject it, or edit it. Nothing is added to the agent's skills without an explicit approval.

**Why this priority**: Human review is an explicit requirement and protects the agent from learning bad or unsafe behavior.

**Independent Test**: Generate a proposal, reject it, and confirm the skills collection is unchanged; generate another, approve it, and confirm it is added.

**Acceptance Scenarios**:

1. **Given** a pending proposal, **When** the user approves it, **Then** the skill is added to the skills collection and the user is told where it was added.
2. **Given** a pending proposal, **When** the user rejects it, **Then** no skill is added and the rejection is recorded so the same proposal is not repeatedly re-suggested.
3. **Given** a pending proposal, **When** the user edits it and then approves, **Then** the edited version is what is added.
4. **Given** no explicit decision (e.g., the session is interrupted), **When** the tool exits, **Then** no skill has been added and the proposal remains available for later review.

---

### User Story 3 - Work with traces from any coding agent (Priority: P2)

A developer using a different coding agent (or a custom one) configures its local trace directory. The tool accepts traces from a variety of agents and formats, and the resulting skill can be installed into the skills location that agent uses.

**Why this priority**: Agent-agnostic support is a stated requirement but delivers value on top of the core flow working for at least one agent.

**Independent Test**: Configure local trace directories for at least two different agents and confirm traces from each yield a proposal without manually supplying or converting individual traces.

**Acceptance Scenarios**:

1. **Given** configured local trace directories containing new or changed traces from two different coding agents, **When** the user chooses a scope that includes them, **Then** the tool reads both without requiring the user to supply or convert individual traces.
2. **Given** a stored trace in a format the tool does not recognize, **When** encountered, **Then** the tool explains which formats are supported and how to provide a generic trace.
3. **Given** the user has not specified another skills location, **When** a skill is approved for this project, **Then** it is written to this project's `.github/skills` directory.
4. **Given** the user specifies another target agent or skills location, **When** a skill is approved, **Then** it is written there in the layout that location expects.

---

### User Story 4 - Avoid duplicate or conflicting skills (Priority: P3)

When a proposed lesson overlaps with an existing skill, the tool tells the user and offers to update the existing skill, keep both, or discard the proposal, instead of silently creating near-duplicates.

**Why this priority**: Keeps the skills collection healthy over time, but is not required for the first useful release.

**Independent Test**: Approve a skill, then run the tool on a similar trace and confirm the overlap is flagged.

**Acceptance Scenarios**:

1. **Given** an existing skill covering the same lesson, **When** a similar proposal is produced, **Then** the user is shown the overlapping skill and offered update, keep both, or discard.

---

### Edge Cases

- Trace is empty, truncated, or corrupted: the tool reports the problem and creates nothing.
- The selected session scope contains no new or changed sessions: the tool reports that there are no sessions to analyze.
- A configured trace directory is missing or unreadable: the tool identifies that location, reports the problem, and continues with other readable locations.
- Trace is very large: the tool still completes, or states clearly what portion was analyzed.
- Trace contains secrets (API keys, tokens, personal data): these are not copied into proposals, and the user is warned if any were detected.
- Multiple distinct lessons exist in one trace: the tool proposes each separately so each can be accepted or rejected independently.
- A skill with the same name already exists in the target location: the tool never overwrites it without explicit confirmation.
- The user runs the tool non-interactively (e.g., in automation): proposals are saved for later review and no skill is added automatically.
- The target skills location does not exist or is not writable: the tool reports this and leaves the approved skill available for retry.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST read coding-agent chat traces from `~/.copilot/session-state` by default; users MUST NOT have to supply individual traces explicitly.
- **FR-018**: At the start of each run, System MUST let the user choose to analyze sessions explicitly associated with the current project across available trace locations or all sessions from those locations; sessions without project association MUST be included only in the all-sessions scope.
- **FR-019**: System MUST analyze only sessions within the selected scope that are new or changed since its previous completed analysis and MUST provide a way to request a full re-scan.
- **FR-020**: Users MUST be able to configure additional local trace directories so traces from other coding agents can be read without individually supplying trace files.
- **FR-002**: System MUST analyze a trace and identify zero or more lessons that generalize beyond the specific task in the trace.
- **FR-003**: System MUST produce, for each lesson, a proposed skill containing a name, a description of when it applies, actionable instructions, and references to the supporting evidence in the trace.
- **FR-004**: System MUST present each proposal to the user for review before any skill is added.
- **FR-005**: Users MUST be able to approve, reject, or edit each proposal independently.
- **FR-006**: System MUST add a skill to the skills collection only after explicit user approval.
- **FR-007**: System MUST report when no generalizable lesson is found and MUST NOT create an empty or low-value proposal in that case.
- **FR-008**: System MUST accept traces from multiple coding agents and formats, including a documented generic format for unsupported agents, from the default or user-configured local trace locations.
- **FR-009**: System MUST use this project's `.github/skills` directory as the default destination for approved skills; users MUST be able to choose another target agent or skills location, and the skill MUST be written in the layout that location expects.
- **FR-010**: Before sending session content to the configured AI service, System MUST screen it for secrets and personal data, remove detected sensitive values, and clearly inform the user that screened session content is sent to that service. System MUST exclude sensitive values from proposed skills and warn the user when any were found.
- **FR-011**: System MUST generalize or omit project-specific details (names, paths, private identifiers) from proposed skills.
- **FR-012**: System MUST NOT overwrite an existing skill without explicit user confirmation.
- **FR-013**: System MUST detect proposals that overlap with existing skills and let the user choose to update, keep both, or discard.
- **FR-014**: System MUST persist undecided proposals and record rejections so they can be reviewed later and not repeatedly re-proposed.
- **FR-015**: System MUST give clear, actionable error messages for empty, malformed, or unsupported traces, and for unwritable target locations.
- **FR-016**: System MUST support non-interactive use that saves proposals for later review without adding any skill.
- **FR-017**: System MUST show the user exactly what will be added (full skill content and destination) before approval takes effect.

### Key Entities

- **Agent Trace**: A recorded session of a coding agent: the sequence of user requests, agent actions, tool results, errors, and outcomes. Attributes include source agent and format.
- **Lesson**: A generalizable insight derived from one or more moments in a trace, with a statement of the situation it applies to and the supporting evidence.
- **Proposed Skill**: A draft skill derived from a lesson: name, applicability description, instructions, evidence references, and review status (pending, approved, rejected).
- **Skill**: An approved, installed unit of reusable agent guidance residing in an agent's skills collection.
- **Review Decision**: The user's approve, reject, or edit outcome for a proposed skill, with time and optional reason.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can go from a trace to a reviewed, approved skill in under 5 minutes for a typical session trace.
- **SC-002**: In 100% of runs, no skill is added to any skills collection without an explicit user approval.
- **SC-003**: On a benchmark set of traces with known lessons, at least 70% of reviewed proposals are judged by the user as accurate and generalizable.
- **SC-004**: Traces from at least three different coding agents can be processed without manual conversion by the user.
- **SC-005**: Zero secrets present in input traces appear in any proposed or installed skill during validation testing.
- **SC-008**: Zero secrets identified by the screening process are sent to the configured AI service during validation testing.
- **SC-006**: In a trial where an agent is given an approved skill and repeats a similar task, the previously observed mistake recurs less often than without the skill.
- **SC-007**: At least 90% of first-time users complete the trace-to-approved-skill flow without consulting documentation beyond the built-in help.

## Assumptions

- `~/.copilot/session-state` is the default trace source; additional local trace directories can be configured for other coding agents. Manual trace capture or export is not required for the primary workflow.
- Session scope is selected by the user on each run, between sessions explicitly associated with the current project and all sessions in the default and configured local trace locations. Sessions without project association are included only in the all-sessions scope.
- A "skill" means a reusable, named piece of instruction guidance with a description of when to use it, following a common folder-and-file convention that coding agents can load.
- Known agents' skills locations and trace formats are supported through built-in knowledge; other agents are supported through a generic trace format and a user-specified skills location.
- Lesson extraction uses an AI model the user has access to; the user provides the necessary access, and screened session content is sent only to that model provider the user configures after clear disclosure.
- Review happens through an interactive terminal flow by default.
- Automatically re-running agents or measuring agent improvement end to end is out of scope; the tool only proposes and installs skills.
- Editing or deleting previously installed skills beyond the update-on-overlap option is out of scope for the first version.
- The tool is used by a single developer on a local machine; team sharing and approval workflows are out of scope.
- The tool can determine whether a session is new or changed since a previous analysis.
- The current project's `.github/skills` directory is the default installation destination; the user can choose a different compatible destination.
