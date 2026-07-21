# Converge — Repository Agent Instructions

This file defines shared rules for every AI agent working in this repository.
Tool-specific instruction files may add narrower guidance but MUST NOT weaken or
contradict these rules.

## Instruction Priority

Follow platform, legal, safety, and security policies first. Within the
repository, apply instructions in this order:

1. `.specify/memory/constitution.md`
2. Approved product and engineering documents in `docs/`
3. Approved feature artifacts under `specs/`
4. This file and applicable tool-specific instruction files
5. The current user request

The current request defines the task but does not override normative project
decisions. If normative sources conflict, stop and report the exact conflict.
Do not silently choose an interpretation or create competing requirements.

Read only the sources relevant to the current task. Prefer targeted sections
over loading the entire documentation set.

## Normative Documents

- `docs/PRODUCT.md` defines durable product direction, priorities, and boundaries.
- `docs/PRD.md` defines functional requirements and delivery scope.
- `docs/MVP.md` defines the approved first-delivery subset and implementation order.
- `docs/DESIGN.md` defines UX, visual language, responsive behavior, and accessibility.
- Approved artifacts in `specs/` define feature-level behavior and acceptance criteria.
- The Project Constitution defines repository-wide non-negotiable practices.

Keep these sources synchronized when an approved decision affects more than one
of them. Do not change product scope, architecture, or normative requirements as
an incidental part of implementation.

## Language and Communication

- Use English for code, identifiers, comments, documentation, specifications,
  test names, and suggested commit messages.
- Communicate with the user in their language; default to Brazilian Portuguese.
- Keep plans, progress updates, and handoffs concise and evidence-based.
- Distinguish facts, assumptions, estimates, inferences, and unresolved risks.
- Never claim a command, check, or test passed unless it was actually executed.

## Approved-Task Autonomy

Within an explicitly approved task, agents may inspect, implement, and validate
without requesting approval for each edit.

Stop and request direction when any of the following applies:

- Normative documents conflict.
- Material ambiguity would change behavior, architecture, or acceptance criteria.
- The work would expand the approved scope or begin an unapproved delivery stage.
- A high-risk, destructive, irreversible, or boundary-crossing action is required.
- Required credentials, authority, artifacts, or human approval are missing.
- Existing user changes cannot be preserved safely.

Use a reasonable, documented assumption only for low-risk implementation details.
Report any assumption that materially affected the result.

## Spec Kit Policy

Use Spec Kit proportionally to risk and scope.

Spec Kit is mandatory before implementation when a change introduces or
materially modifies:

- User-visible behavior or acceptance criteria.
- Product scope, priorities, or supported journeys.
- Domain models or application use cases.
- Public contracts, IPC, HTTP, WebSocket, or provider interfaces.
- Persistence schemas, SQLx migrations, or event formats.
- Architecture, trust, security, privacy, or permission boundaries.
- A delivery slice or cross-module capability.

Small documentation corrections, formatting-only changes, dependency upkeep,
and narrow behavior-preserving refactors may use a proportional workflow. Bug
fixes require a clear reproduction and risk-appropriate tests. If a small task
crosses any boundary above, stop and switch to Spec Kit before continuing.

For Spec Kit work:

1. Confirm the required artifacts exist and are approved.
2. Verify explicit Constitution compliance in the specification, plan, and tasks.
3. Reuse existing artifacts instead of creating competing versions.
4. Resolve material decisions through concise Q&A before implementation.
5. Preserve stable requirement IDs and end-to-end traceability.
6. Do not implement until scope and required artifacts are approved.

## Standard Workflow

1. Inspect repository state, relevant instructions, and existing user changes.
2. Classify the task by scope and risk; determine whether Spec Kit is required.
3. Identify the smallest complete change that satisfies the approved scope.
4. For non-trivial work, state a short plan and evaluate parallel delegation.
5. Implement without unrelated edits and within established module boundaries.
6. Run narrow checks first, then broader validation proportional to risk.
7. Perform an explicit Constitution compliance review.
8. Report changed files, validation, remaining risks, and manual Git commands.

Use the repository `justfile` as the command facade. Inspect available recipes
with `just --list`; do not invent recipe names. Use direct Cargo or pnpm commands
only when no suitable recipe exists or while diagnosing the recipe itself.

## Parallel Delegation

For non-trivial tasks, evaluate whether independent subtasks can run in parallel.
Delegation is a tool for reducing latency and improving review quality, not a
default requirement.

Parallel delegation SHOULD be used when two or more subtasks are:

- Independent and clearly bounded.
- Expected to require meaningful investigation, implementation, or validation.
- Able to produce independently reviewable results.
- Assigned non-overlapping file ownership when edits are involved.

Good candidates include:

- Exploring separate modules or architectural layers.
- Investigating competing debugging hypotheses.
- Researching frontend, backend, contracts, or persistence independently.
- Reviewing security, accessibility, performance, and tests independently.
- Performing adversarial review or independent validation after implementation.

Parallel delegation SHOULD NOT be used when:

- The task is small or faster to complete directly.
- One subtask depends on another worker's unfinished result.
- Workers would edit the same files or tightly coupled code.
- Scope or acceptance criteria are materially ambiguous.
- Coordination overhead would exceed the expected benefit.

### Concurrency and Ownership

- Prefer 2 to 3 parallel workers at a time.
- Use more only when scopes are genuinely independent and ownership is explicit.
- Assign exact files, directories, or read-only responsibilities to each worker.
- Shared foundational files MUST have a single editing owner.
- Workers MUST NOT modify files outside their assignment without coordinator approval.
- Recursive delegation is disabled by default; the coordinator must authorize it
  explicitly without exceeding the agreed concurrency boundary.
- Agents MUST NOT create, switch, remove, or manage Git worktrees automatically.
  Worktrees may be used only when prepared or explicitly approved by the human
  maintainer.

### Coordinator Responsibilities

The coordinating agent MUST:

1. Define each worker's scope, constraints, expected output, and file ownership.
2. Provide the context required for the assignment without unnecessary expansion.
3. Prevent duplicated work and conflicting edits.
4. Retain responsibility for product and architectural decisions.
5. Review, reconcile, and validate every delegated result.
6. Run integrated checks after combining the work.
7. Report what was delegated and how the combined result was validated.

Delegated agents inherit every repository instruction, security boundary, testing
requirement, approval gate, and Git restriction. They MUST NOT expand scope or
perform repository-publishing actions.

## Repository Structure and Boundaries

Follow the established layout:

- `apps/web/` — React, TypeScript, and Vite interface.
- `crates/` — Rust modules organized around Domain, Application, and Adapters.
- `packages/` — shared or generated TypeScript contracts.
- `migrations/` — versioned, forward-only SQLx migrations.
- `tests/` — contract, integration, security, and end-to-end tests.
- `docs/` — approved product, architecture, design, and operational documents.
- `specs/` — approved Spec Kit feature artifacts.

Do not introduce a new top-level structure or move responsibilities across
layers without an approved architectural decision.

## Architecture Guardrails

- Preserve Clean Architecture inside the modular monolith.
- Dependencies point inward: Interface and Adapters depend on Application;
  Application depends on Domain; Domain remains independent.
- Domain and Application MUST NOT depend on React, HTTP, WebSocket, Tauri, SQLx,
  filesystem, PTY, or provider-specific implementations.
- Keep provider-specific behavior behind common Rust ports and adapters.
- Preserve Claude and Codex parity unless an approved requirement documents a
  provider-specific difference.
- Keep React server state in TanStack Query and transient UI/layout state in Zustand.
- Use typed contracts at Rust-to-TypeScript boundaries; do not duplicate contract
  shapes manually when generation is available.
- Keep immutable normalized events as the ledger and aggregates reconstructable.
- Treat YAML and Markdown project files as portable authoritative sources where
  defined by the product documents.

## Security and Data Integrity

- The web interface MUST NOT receive generic shell or filesystem access.
- Expose narrow, typed Rust operations with validated inputs, capabilities, and scopes.
- Preserve loopback-only service binding, random ports, ephemeral authentication,
  Host/Origin validation, and closed CORS where applicable.
- Treat the canonical project path as the trust boundary.
- Block external symlink traversal unless explicitly authorized by the user.
- Never persist provider credentials; keep approved reads in memory and redact secrets.
- Use atomic writes, conflict detection, and approved backups for editable files.
- Do not introduce arbitrary shell execution, unrestricted network access, remote
  WebView content, telemetry by default, or silent diagnostic uploads.
- Original provider payloads remain disabled unless an approved requirement changes
  that policy; normalized metadata must not leak prompts, outputs, or secrets.
- Never weaken protections against secret leakage, data corruption, or trusted
  directory escape to simplify implementation or testing.

## Frontend and Design

- Follow `docs/DESIGN.md` and the Graphite Signal design system.
- Use approved semantic design tokens instead of raw visual values.
- Treat WCAG 2.2 AA, keyboard navigation, visible focus, reduced motion, and
  non-color state communication as acceptance criteria.
- Implement loading, empty, error, offline, stale, disabled, and unsupported
  states where relevant.
- Preserve progressive disclosure and keep terminal views read-only.
- Do not introduce generic dashboard patterns that contradict the approved
  information architecture or visual direction.

## Testing and Quality

Testing is mandatory according to risk.

Use Red-Green-Refactor for:

- Domain logic.
- Application use cases.
- Contracts.
- Security boundaries.
- Bug fixes.

UI, configuration, infrastructure, and adapters require appropriate tests in
the same change but do not require strict test-first ordering.

Use the applicable test layers:

- Rust unit and integration tests with redacted Claude and Codex fixtures.
- Contract tests for HTTP, WebSocket, IPC, and generated TypeScript boundaries.
- Vitest and React Testing Library for frontend behavior.
- SQLx migration and aggregate-rebuild tests.
- Security tests for authorization, trust boundaries, paths, origins, and secrets.
- Browser E2E tests for integrated user journeys on supported platforms.

Every delivery slice MUST finish demonstrable, integrated into the accumulated
journey, and covered by appropriate white-box and black-box tests before the next
slice begins.

When a check cannot run, report the exact blocker and what remains unverified.
Do not weaken or delete tests merely to obtain a passing result.

## Git and Release Boundaries

Agents may use read-only Git inspection such as `git status`, `git diff`,
`git log`, and `git show`.

Agents, integrations, delegated workers, and Spec Kit workflows MUST NOT execute:

- `git add`, `commit`, `push`, `merge`, `tag`, `rebase`, or `reset`.
- Pull-request creation or equivalent repository-publishing actions.
- Commands that rewrite history, discard user changes, or publish repository state.

The human maintainer exclusively controls staging, commits, pushes, merges, tags,
and pull requests. At handoff, agents may suggest precise commands for the human
to review and run manually.

CI/CD automation MAY build, test, sign, checksum, and publish releases only when
triggered by an approved semantic-version tag created manually by the human
maintainer.

## Completion Criteria

A task is complete only when:

- Approved scope and acceptance criteria are satisfied.
- Applicable tests and quality checks pass, or exact blockers are reported.
- Security, privacy, accessibility, and data-integrity impacts were reviewed.
- Required documentation and Spec Kit artifacts remain synchronized.
- Delegated work, if any, was reviewed and validated as an integrated result.
- No unrelated user changes were overwritten.
- The final report includes an explicit Constitution compliance result.

Never hide uncertainty, skipped validation, unsupported behavior, or known risk.
