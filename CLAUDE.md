# Converge — Claude Code Instructions

@AGENTS.md

This file defines Claude Code-specific behavior. Shared project rules belong in
`AGENTS.md`; product, engineering, and visual decisions remain in the normative
documents referenced below.

## Instruction Priority

Follow applicable platform and safety policies first. Within the repository,
apply instructions in this order:

1. `.specify/memory/constitution.md`
2. Approved product and engineering documents in `docs/`
3. Approved feature artifacts under `specs/`
4. `AGENTS.md` and this Claude-specific overlay
5. The current user request

When normative sources conflict, stop and report the exact conflict. Do not
silently choose one interpretation or implement around it.

At the start of a task, read only the sources relevant to that task. Do not load
the full documentation set when targeted sections are sufficient.

## Language

- Use English for code, identifiers, comments, documentation, specifications,
  test names, and suggested commit messages.
- Communicate with the user in their language; default to Brazilian Portuguese.
- Keep progress updates and final reports concise and evidence-based.

## Autonomy

Within an explicitly approved task, inspect, implement, and validate without
requesting approval for each edit.

Stop and ask for direction only when at least one of these conditions applies:

- Normative documents conflict.
- Material ambiguity would change product behavior or architecture.
- The work requires expanding the approved scope.
- A high-risk, destructive, irreversible, or boundary-crossing action is needed.
- Required credentials, authority, artifacts, or human approval are missing.
- Existing user changes cannot be preserved safely.

Prefer a reasonable, documented assumption for low-risk implementation details.
State the assumption in the final report when it affects the result.

## Spec Kit Policy

Use Spec Kit proportionally to risk and scope.

Spec Kit is mandatory before implementation when a change introduces or
materially modifies any of the following:

- User-visible functional behavior or acceptance criteria.
- Product scope, priorities, or supported journeys.
- Domain models or application use cases.
- Public contracts, IPC, HTTP, WebSocket, or provider interfaces.
- Persistence schemas, migrations, or event formats.
- Architecture, trust, security, privacy, or permission boundaries.
- A delivery slice or a cross-module capability.

Small documentation corrections, formatting-only changes, dependency upkeep,
and narrow refactors that preserve behavior may use a proportional workflow.
Bug fixes still require a clear reproduction and risk-appropriate tests. If a
supposedly small change crosses any boundary above, switch to Spec Kit before
continuing.

For Spec Kit work:

1. Verify that the Constitution compliance check is present.
2. Reuse approved specifications, plans, and task lists; do not create competing
   artifacts for the same scope.
3. Resolve open decisions through concise Q&A before implementation.
4. Keep requirement IDs and traceability intact.
5. Do not implement until the required artifacts and scope are approved.
6. Update affected normative documents in the same change when a decision alters
   them.

## Execution Workflow

1. Inspect repository state and relevant instructions before editing.
2. Identify the smallest complete change that satisfies the approved task.
3. State a short plan for non-trivial work.
4. Preserve existing user changes and avoid unrelated edits.
5. Implement within the established architecture and module boundaries.
6. Run the narrowest relevant checks first, then broader checks proportional to
   risk.
7. Perform an explicit Constitution compliance review.
8. Report the outcome, changed files, validation performed, remaining risks, and
   suggested manual Git commands.

Use the repository `justfile` as the command facade. Inspect it with
`just --list` rather than inventing commands. Use direct Cargo or pnpm commands
only when no suitable recipe exists or when diagnosing the recipe itself.

## Testing

Testing is mandatory according to risk.

Use Red-Green-Refactor for:

- Domain logic.
- Application use cases.
- Contracts.
- Security boundaries.
- Bug fixes.

UI, configuration, infrastructure, and adapters require appropriate tests in
the same change, but do not require strict test-first ordering.

For affected areas, run applicable formatting, linting, unit, integration,
contract, frontend, migration, security, and E2E checks. Do not claim a check
passed unless it was executed successfully. If a check cannot run, report the
exact blocker and what remains unverified.

## Architecture and Security Guardrails

- Preserve Clean Architecture inside the modular monolith: Domain, Application,
  Adapters, and Interface dependencies point inward.
- Keep the React frontend outside generic shell and filesystem access.
- Expose narrow, typed Rust operations with validated arguments and scopes.
- Treat canonical project boundaries, symlink handling, provider credentials,
  secret redaction, atomic writes, conflict detection, and reconstructable data
  as non-negotiable protections.
- Never weaken protections against secret leakage, data corruption, or trusted
  directory escape.
- Do not introduce remote WebView content, telemetry by default, unrestricted
  network access, or arbitrary shell execution.
- Preserve Claude and Codex parity unless an approved requirement explicitly
  documents a provider-specific difference.

For frontend work, follow `docs/DESIGN.md`, use design tokens rather than raw
visual values, and treat WCAG 2.2 AA, keyboard access, visible focus, reduced
motion, and non-color state communication as acceptance criteria.

## Git and Release Boundaries

Read-only Git inspection is allowed, including `git status`, `git diff`,
`git log`, and `git show`.

Do not execute repository-publishing or history-changing actions, including:

- `git add`, `commit`, `push`, `merge`, `tag`, `rebase`, or `reset`.
- Pull-request creation or equivalent publishing actions.
- Rewriting or discarding the user's working tree changes.

At handoff, provide precise Git commands for the human maintainer to review and
run manually. CI/CD may build, test, sign, checksum, and publish a release only
after a human maintainer creates the approved semantic-version tag.

## Completion Criteria

A task is complete only when:

- The approved scope and acceptance criteria are satisfied.
- Applicable tests and quality checks pass, or blockers are explicitly reported.
- Security, privacy, accessibility, and data-integrity impacts were reviewed.
- Required documentation and Spec Kit artifacts remain synchronized.
- No unrelated user changes were overwritten.
- The final report includes the Constitution compliance result.

Never hide uncertainty, skipped validation, unsupported behavior, or known risk.

## Subagents

Follow the parallel-delegation policy defined in `AGENTS.md`.

When that policy recommends delegation, Claude Code SHOULD use native subagents
for independent research, implementation, testing, or review tasks.

Do not create parallel workers for small, sequential, ambiguous, or overlapping
work. The coordinating agent remains responsible for integration and validation.
