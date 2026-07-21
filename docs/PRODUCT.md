# Converge — Product Definition

**Version:** 1.0.0  
**Status:** Draft for approval  
**Last updated:** 2026-07-21

## 1. Document Role

This document defines Converge's durable product direction: its purpose, audience,
problems, value proposition, principles, capability boundaries, and success outcomes.

It does not define detailed functional requirements, implementation architecture,
visual specifications, or slice-level acceptance criteria.

- `PRODUCT.md` governs product vision, positioning, priorities, and boundaries.
- `PRD.md` governs detailed product requirements and delivery scope.
- `DESIGN.md` governs user experience, visual language, and accessibility.
- The Project Constitution governs project-wide practices and constraints.

A conflict between normative documents MUST block implementation until resolved.

## 2. Product Statement

Converge is a local-first control center for individual developers who work with
multiple coding agents.

It brings subscription usage, estimated costs, agents, sessions, execution progress,
workflows, and context files into one coherent interface while preserving explicit
human control.

## 3. Vision

Enable developers to understand and coordinate agent-assisted development without
reconstructing project state across disconnected terminals, provider interfaces,
session logs, and orchestration files.

Converge should become the trusted operational layer between the developer, local
projects, and coding-agent providers.

It complements existing agent CLIs and system terminals. It does not replace them.

## 4. Primary Audience

The initial audience is individual developers who:

- Use Claude Code, Codex CLI, or both.
- Work with multiple agents or sessions.
- Need visibility into subscription quotas and consumption.
- Value local control, privacy, transparency, and inspectable files.
- Are comfortable with terminals, Git, Markdown, and configuration files.
- May use Spec Kit, but must not depend on it to benefit from Converge.

Team collaboration is a possible future direction, not part of the current audience.

## 5. Core Problems

Converge addresses three problems in priority order:

1. Developers cannot easily understand limits, consumption, renewal windows, and
   estimated costs across coding-agent subscriptions.
2. Agent activity is fragmented across terminals, sessions, and provider-specific
   interfaces.
3. Context, workflow, and orchestration files are disconnected from the executions
   they govern.

.

This fragmentation increases context switching and makes execution state, progress,
failures, and resource consumption harder to understand.

## 6. Core Jobs to Be Done

Converge helps the user:

1. Understand current subscription usage and when limits reset.
2. Attribute token consumption and API-equivalent estimated cost to providers,
   models, agents, sessions, and workflow steps.
3. Follow multiple agents without constantly switching terminals.
4. Understand the current phase and progress of an Agent Loop.
5. Manage relevant context and orchestration files from the same workspace.
6. Start, resume, stop, and coordinate agent activity through safe, explicit controls.
7. Continue inspecting local history and files when remote services are unavailable.

## 7. Value Proposition

Converge reduces operational fragmentation by combining:

- Subscription and usage observability.
- Agent and session visibility.
- Agent Loop progress.
- Workflow orchestration.
- Context-file management.
- Project trust and safe execution controls.

The result should be a clearer understanding of what agents are doing, how resources
are being consumed, which sources produced the displayed information, and when human
attention is required.

## 8. Product Principles

### Local First

Project data, history, settings, and control should remain on the user's machine
whenever possible.

### Human Controlled

Relevant actions must originate from explicit user intent. Risky, destructive, or
boundary-crossing operations require proportionate confirmation.

### Observable

Execution state, progress, failures, stale data, unsupported data, and recovery
options must be visible.

### Honest About Uncertainty

Provider-reported, locally observed, inferred, estimated, stale, and unavailable
information must be clearly distinguishable.

### Agent Oriented

The product should organize work around named agents, sessions, and responsibilities
rather than forcing the user into provider-specific mental models.

### Safe by Default

New projects begin restricted. Arbitrary terminal input, unrestricted shell access,
implicit trust, and silent external-path access are outside the default experience.

### Progressive Disclosure

The interface should present a useful summary first and reveal sessions, turns,
commands, tool calls, and usage details on demand.

### Files Remain Portable

Shareable workflows and context belong in human-readable project files. Internal
persistence must not replace those files as the project's portable source of truth.

### Optional Integrations

Spec Kit and future integrations may enrich the experience but must not become
requirements for the core product.

### Focused Scope

Converge operates on one active local project at a time and prioritizes depth,
clarity, and safety over broad workspace management.

## 9. Product Capabilities

The complete product direction contains six capability pillars:

### Analytics

Subscription quota visibility, token consumption, renewal windows, trends, warnings,
and API-equivalent cost estimates clearly separated from actual billing.

### Agents and Sessions

Named agents, concurrent sessions, lifecycle controls, externally detected sessions,
and read-only terminal views.

### Agent Loops

Visual representation of phases, turns, commands, tool calls, progress, failures,
and consumption.

### Workflows

Safe orchestration of sequential, parallel, and collaborative agent work using
portable workflow definitions.

### Context Files

Discovery, inspection, preview, and safe editing of recognized project context and
orchestration files.

### Projects and Trust

Explicit folder selection, restricted initial state, visible capability boundaries,
and deliberate trust decisions.

## 10. Delivery Direction

The first delivery is a locally executed, web-based vertical slice covering the
minimum complete journey:

1. Select and trust a project.
2. Detect Claude and Codex.
3. Import and inspect analytics.
4. Start and observe managed sessions.
5. View Agent Loop progress.
6. Inspect and safely edit supported context files.
7. Access persisted local information while offline.

This first delivery is a subset of the complete product direction. Deferred
capabilities remain part of the project unless explicitly removed through the
approved product-decision process.

The product may later expand to additional compiled provider adapters and,
eventually, a controlled dynamic-plugin model.

## 11. Current Product Boundaries

Converge is not currently intended to provide:

- Cloud synchronization or multi-user collaboration.
- Team-level aggregation.
- API billing management.
- Generic arbitrary terminal input through the web interface.
- More than one active project folder at a time.
- Automatic installation of agent providers.
- Provider credential management or modification.
- Support for providers other than Claude and Codex.
- Dynamic plugins.
- Default background operation or system-tray behavior.
- Remote content executing inside the product interface.
- Telemetry or crash reporting enabled by default.
- Automatic blocking based only on quota warnings.

These boundaries may change only through an explicit product decision and corresponding
updates to normative documents.

## 12. Success Outcomes

Converge is successful when the user can:

- Understand current consumption, quota state, renewal timing, and estimated cost
  without manually combining multiple sources.
- Follow several agents and sessions from one interface.
- Identify the current state and progress of an execution.
- Distinguish reported facts from estimates, inference, stale data, and unsupported
  data.
- Manage relevant context files without silent overwrites or loss of external changes.
- Use the product without surrendering control of project access or agent actions.
- Continue using core local capabilities without Spec Kit.
- Switch less frequently between terminals, provider interfaces, logs, and files to
  understand the state of the work.

Quantitative adoption and performance targets should be established from real usage
data rather than invented before a reliable baseline exists.

## 13. Product Decision Test

A proposed capability belongs in Converge when it materially improves at least one
of the following:

- Understanding agent consumption or limits.
- Observing agents, sessions, and execution progress.
- Coordinating safe agent workflows.
- Managing the context that governs agent behavior.
- Preserving human control, transparency, privacy, or recoverability.

Capabilities that do not support these outcomes should be deferred, rejected, or
treated as separate products.

## 14. References

- [Product Discovery](https://app.notion.com/p/39f16378a61d81d0b944cf9ab519b2d6)
- [Vision & Roadmap](https://app.notion.com/p/39f16378a61d81c280c3cbd24d51d2f1)
- [Product Brainstorm](https://app.notion.com/p/39f16378a61d8178979ac12b7f36f0cc)
- [Product Requirements Document](https://app.notion.com/p/39f16378a61d81b2b59dc09df4287481)
- [MVP Definition & Prioritization](https://app.notion.com/p/39f16378a61d81d29689f1eeb3320341)
- [Information Architecture & Screens](https://app.notion.com/p/3a316378a61d81ca93b4e7b7c19a01fc)
