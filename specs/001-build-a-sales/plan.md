
# Implementation Plan: Sales Account Status Web Application

**Branch**: `001-build-a-sales` | **Date**: 2025-09-29 | **Spec**: [/workspaces/AccountTrack/specs/001-build-a-sales/spec.md](/workspaces/AccountTrack/specs/001-build-a-sales/spec.md)
**Input**: Feature specification from `/specs/001-build-a-sales/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from file system structure or context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Fill the Constitution Check section based on the content of the constitution document.
4. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
5. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
6. Execute Phase 1 → contracts, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, `GEMINI.md` for Gemini CLI, `QWEN.md` for Qwen Code or `AGENTS.md` for opencode).
7. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
8. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
9. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary
Build a secure, accessible sales account tracking platform that delivers a responsive React SPA backed by a NestJS Azure Functions API. The system manages account lifecycle data in Azure Cosmos DB (Core SQL API) with change feed support for history timelines, enforces a linear status pipeline, and integrates with Azure AD for MFA-protected access. Observability, GDPR-grade privacy controls, and 90% automated test coverage are mandatory per the constitution.

## Technical Context
**Language/Version**: TypeScript 5.x (frontend & backend), Node.js 20 LTS
**Primary Dependencies**: React 18 + Vite, Chakra UI, React Query, NestJS 10, Azure Functions v4, @azure/cosmos SDK, Azure Identity, MSAL.js, Winston, Playwright
**Storage**: Azure Cosmos DB (Core SQL API), partition key `accountId`, autoscale up to 10,000 RU/s, change feed for history projections; Azure Blob Storage for exports
**Testing**: Jest + React Testing Library, Jest + Supertest, Playwright E2E, k6 smoke
**Target Platform**: Azure Static Web Apps (frontend) + Azure Functions (backend) with Managed Identity; modern browsers (Chromium, Firefox, Safari), internal corporate network
**Project Type**: Web (discrete frontend + backend services)
**Performance Goals**: <2s p95 for primary UI workflows; <200ms backend processing for CRUD/status transitions; search results <500ms for 95th percentile
**Constraints**: Enforce MFA and RBAC via Azure AD; WCAG 2.1 AA UI; GDPR-compliant retention (5 years standard, 30-day erasure SLA); rate limiting 100 req/min per user; 99.5% availability during business hours
**Scale/Scope**: 20k active accounts, 200k history entries/day, 2k concurrent users across sales teams, three role tiers (rep, manager, admin)

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- ✅ Principle 1 (Account Lifecycle Ownership): Enforces linear pipeline, immutable history via change feed, alerts for overdue follow-ups delivered through dashboard widgets.
- ✅ Principle 2 (Security & Privacy Safeguards): Azure AD MFA, AES-256 at rest in Cosmos, TLS 1.3 via Front Door, Key Vault–managed secrets, GDPR retention & erasure workflow documented.
- ✅ Principle 3 (Responsive Performance): SPA optimized with React Query caching, serverless functions targets <200ms, k6 smoke ensures budgets.
- ✅ Principle 4 (Type-Safe Quality Discipline): Full-stack TypeScript strict mode, ESLint + Prettier in CI, ≥90% coverage requirement baked into pipelines.
- ✅ Principle 5 (Inclusive Access & Compliance): Chakra UI accessible primitives, keyboard flows tested via Playwright + axe, alternative text for charts.
- ✅ Platform & Architecture Standards: Azure Static Web Apps + Functions + Cosmos + Key Vault + Application Insights; IaC via Bicep.
- ✅ Development Workflow & Quality Gates: Plan mandates constitution alignment in reviews and CI gating (lint/test/perf/accessibility).

**Initial Constitution Check: PASS**

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── common/
│   │   ├── decorators/
│   │   └── filters/
│   ├── modules/
│   │   ├── accounts/
│   │   │   ├── controllers/
│   │   │   ├── dtos/
│   │   │   ├── services/
│   │   │   └── repositories/
│   │   ├── auth/
│   │   └── exports/
│   └── infrastructure/
│       ├── cosmos/
│       ├── identity/
│       └── telemetry/
└── tests/
   ├── contract/
   ├── integration/
   └── unit/

frontend/
├── src/
│   ├── app/
│   ├── components/
│   ├── pages/
│   ├── hooks/
│   ├── features/accounts/
│   ├── providers/
│   └── routing/
└── tests/
   ├── unit/
   ├── integration/
   └── accessibility/

shared/
├── types/
└── utils/

infrastructure/
├── bicep/
└── github-actions/
```

**Structure Decision**: Dual-repo layout with `frontend/` React SPA and `backend/` NestJS Azure Functions service, plus shared TypeScript types and IaC manifests. Mirrors constitution requirements for clear separation, contract-focused modules, and test directories per layer.

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - Confirm Cosmos DB RU budgeting, autoscale thresholds, and failover strategy.
   - Validate Azure AD authentication flow (interactive + conditional access) for internal sales teams.
   - Define audit trail export limits (date window, file size) aligned with legal/compliance input.
   - Document GDPR retention timeline, erasure workflow, and data residency mapping.
   - Establish offline/throttling strategy when Cosmos DB returns 429s, including queue/backoff behavior.
   - Capture observability signals (logs, metrics, alerts) required by Security and Ops.
   - Enumerate accessibility tooling and regression checks (axe-core integration, keyboard audits).

2. **Generate and dispatch research agents**:
   ```
    For each unknown in Technical Context:
       Task: "Research {unknown} for sales account tracker"
    For each technology choice:
       Task: "Collect best practices for {tech} in Azure serverless web apps"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Extract entities from feature spec** → `data-model.md`:
   - Account, AccountStatus, AccountHistory, UserRole, AuditExportRequest, NotificationSubscription.
   - Include partition key selection, composite indexes, retention/soft-delete metadata.
   - Encode linear status transition matrix and guard rails.

2. **Generate API contracts** from functional requirements:
   - Define REST endpoints: dashboard listings, detail retrieval, status transition mutation, history feed, search, audit export initiation/download, admin configuration.
   - Document auth scopes per role and error models (validation, throttling, auth failures).
   - Publish OpenAPI schemas in `/contracts/accounts.openapi.yaml` and `/contracts/audit.openapi.yaml`.

3. **Generate contract tests** from contracts:
   - Create failing contract tests in `backend/tests/contract` mirroring OpenAPI definitions using Supertest.
   - Include negative tests for forbidden transitions, missing MFA claims, and rate limit responses.

4. **Extract test scenarios** from user stories:
   - Map acceptance criteria to Playwright journeys (filter dashboard, update status with comment, export audit trail, global search).
   - Define API integration tests for history creation, audit export job queueing, Cosmos throttling backoff.

5. **Update agent file incrementally** (O(1) operation):
   - Run `.specify/scripts/bash/update-agent-context.sh copilot`
     **IMPORTANT**: Execute it exactly as specified above. Do not add or remove any arguments.
   - If exists: Add only NEW tech from current plan
   - Preserve manual additions between markers
   - Update recent changes (keep last 3)
   - Keep under 150 lines for token efficiency
   - Output to repository root

**Output**: data-model.md, /contracts/*, failing tests, quickstart.md, agent-specific file

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

**Task Generation Strategy**:
- Load `.specify/templates/tasks-template.md` as base
- Generate tasks from Phase 1 design docs (contracts, data model, quickstart)
- Each contract → contract test task [P]
- Each entity → model creation task [P] 
- Each user story → integration test task
- Implementation tasks to make tests pass

**Ordering Strategy**:
- TDD order: Tests before implementation 
- Dependency order: Models before services before UI
- Mark [P] for parallel execution (independent files)

**Estimated Output**: 25-30 numbered, ordered tasks in tasks.md

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command)
- [x] Phase 1: Design complete (/plan command)
- [ ] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved
- [ ] Complexity deviations documented

---
*Based on Constitution v1.1.0 - See `.specify/memory/constitution.md`*

---
*Based on Constitution v1.1.0 - See `.specify/memory/constitution.md`*
