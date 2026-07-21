---

description: "Task list template for feature implementation"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Tests are mandatory according to Constitution Principle VI and must be selected according to risk. Red-Green-Refactor is mandatory for domain logic, application use cases, contracts, security boundaries, and bug fixes. UI, configuration, infrastructure, and adapters must include appropriate tests in the same change, without requiring strict test-first ordering.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Rust domain/application/adapters**: `crates/[crate-name]/src/`
- **React/TypeScript UI**: `apps/web/`
- **Generated TypeScript contracts**: `packages/`
- **SQLx migrations**: `migrations/`
- **Tests (contract, integration, E2E)**: `tests/`
- Paths shown below are illustrative — adjust to the concrete crates/apps/packages captured in plan.md

<!--
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.

  The /speckit-tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Feature requirements from plan.md
  - Entities from data-model.md
  - Endpoints from contracts/

  Tasks MUST be organized by user story so each story can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment

  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project structure per implementation plan
- [ ] T002 Initialize [language] project with [framework] dependencies
- [ ] T003 [P] Configure linting and formatting tools
- [ ] T004 Verify Constitution compliance per .specify/memory/constitution.md (record findings in plan.md Constitution Check)

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

Examples of foundational tasks (adjust based on your project):

- [ ] T005 Setup database schema and migrations framework in migrations/
- [ ] T006 [P] Implement authentication/authorization framework in crates/[crate-name]/src/
- [ ] T007 [P] Setup API/command routing and middleware structure in crates/[crate-name]/src/
- [ ] T008 Create base domain models/entities that all stories depend on in crates/[crate-name]/src/domain/
- [ ] T009 Configure error handling and logging infrastructure in crates/[crate-name]/src/
- [ ] T010 Setup environment configuration management in crates/[crate-name]/src/

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 1 (MANDATORY — scope per risk, Constitution Principle VI) ⚠️

> Tests are mandatory according to Constitution Principle VI and must be selected according to risk.
> Red-Green-Refactor is mandatory for domain logic, application use cases, contracts, security boundaries, and bug fixes.
> UI, configuration, infrastructure, and adapters must include appropriate tests in the same change, without requiring strict test-first ordering.

- [ ] T011 [P] [US1] Contract test for [endpoint] in tests/contract/[name].rs
- [ ] T012 [P] [US1] Integration test for [user journey] in tests/integration/[name].rs

### Implementation for User Story 1

- [ ] T013 [P] [US1] Create [Entity1] model in crates/[crate-name]/src/domain/[entity1].rs
- [ ] T014 [P] [US1] Create [Entity2] model in crates/[crate-name]/src/domain/[entity2].rs
- [ ] T015 [US1] Implement [Service] in crates/[crate-name]/src/application/[service].rs (depends on T013, T014)
- [ ] T016 [US1] Implement [endpoint/feature] in crates/[crate-name]/src/adapters/[file].rs or apps/web/src/[location]/[file].tsx
- [ ] T017 [US1] Add validation and error handling
- [ ] T018 [US1] Add logging for user story 1 operations

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 2 (MANDATORY — scope per risk, Constitution Principle VI) ⚠️

- [ ] T019 [P] [US2] Contract test for [endpoint] in tests/contract/[name].rs
- [ ] T020 [P] [US2] Integration test for [user journey] in tests/integration/[name].rs

### Implementation for User Story 2

- [ ] T021 [P] [US2] Create [Entity] model in crates/[crate-name]/src/domain/[entity].rs
- [ ] T022 [US2] Implement [Service] in crates/[crate-name]/src/application/[service].rs
- [ ] T023 [US2] Implement [endpoint/feature] in crates/[crate-name]/src/adapters/[file].rs or apps/web/src/[location]/[file].tsx
- [ ] T024 [US2] Integrate with User Story 1 components (if needed)

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 3 (MANDATORY — scope per risk, Constitution Principle VI) ⚠️

- [ ] T025 [P] [US3] Contract test for [endpoint] in tests/contract/[name].rs
- [ ] T026 [P] [US3] Integration test for [user journey] in tests/integration/[name].rs

### Implementation for User Story 3

- [ ] T027 [P] [US3] Create [Entity] model in crates/[crate-name]/src/domain/[entity].rs
- [ ] T028 [US3] Implement [Service] in crates/[crate-name]/src/application/[service].rs
- [ ] T029 [US3] Implement [endpoint/feature] in crates/[crate-name]/src/adapters/[file].rs or apps/web/src/[location]/[file].tsx

**Checkpoint**: All user stories should now be independently functional

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] TXXX [P] Documentation updates in docs/
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all stories
- [ ] TXXX [P] Additional unit tests per Constitution Principle VI in tests/unit/ or colocated with source
- [ ] TXXX Security hardening
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Per Constitution Principle VI: tests for domain logic, application use cases, contracts, and security boundaries MUST be written and FAIL before implementation (Red-Green-Refactor); tests for UI, configuration, infrastructure, and adapters MUST be included in the same change without requiring test-first ordering
- Models before services
- Services before endpoints
- Core implementation before integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all tests for User Story 1 together:
Task: "Contract test for [endpoint] in tests/contract/[name].rs"
Task: "Integration test for [user journey] in tests/integration/[name].rs"

# Launch all models for User Story 1 together:
Task: "Create [Entity1] model in crates/[crate-name]/src/domain/[entity1].rs"
Task: "Create [Entity2] model in crates/[crate-name]/src/domain/[entity2].rs"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1
   - Developer B: User Story 2
   - Developer C: User Story 3
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify tests fail before implementing domain logic, application use cases, contracts, and security boundaries (Red-Green-Refactor); other tests MUST still be included in the same change per Constitution Principle VI
- Per Constitution Principle X (Human-Controlled Version Control), agents MUST NOT stage, commit, or push after tasks; report a change summary, files changed, verification results, risks, and exact suggested Git commands for the human maintainer instead
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
