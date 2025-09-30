# Tasks: Sales Account Status Web Application

**Input**: Design documents from `/specs/001-build-a-sales/`
**Prerequisites**: `plan.md`, `research.md`, `data-model.md`, `quickstart.md`, `contracts/`

## Phase 1: Setup
- [ ] T001 Initialize npm workspaces in `package.json` with `frontend`, `backend`, and `shared` packages plus `tsconfig.base.json` for strict TypeScript settings.
- [ ] T002 Configure repository linting/formatting via `.eslintrc.cjs`, `.prettierrc.cjs`, `.lintstagedrc.cjs`, and Husky hooks under `.husky/` enforcing constitution gates.
- [ ] T003 Scaffold Azure Functions NestJS backend with `backend/package.json`, `backend/tsconfig.json`, `backend/host.json`, and bootstrap files `backend/src/main.ts` & `backend/src/app.module.ts`.
- [ ] T004 Scaffold React + Vite + Chakra frontend with `frontend/package.json`, `frontend/vite.config.ts`, `frontend/src/main.tsx`, and `frontend/src/app/App.tsx` wired for React Router.
- [ ] T005 Establish shared types workspace at `shared/` including `shared/package.json`, `shared/tsconfig.json`, and initial domain contracts in `shared/types/index.ts`.

## Phase 2: Tests First (TDD)
- [ ] T006 [P] Add failing Sales Account API contract tests covering GET `/accounts`, GET `/accounts/{accountId}`, GET `/accounts/search`, POST `/accounts/{accountId}/status`, and GET `/accounts/{accountId}/history` in `backend/tests/contract/accounts/accounts.contract.spec.ts`.
- [ ] T007 [P] Add failing Audit Export API contract tests for POST `/accounts/{accountId}/audit-exports` and GET `/accounts/{accountId}/audit-exports/{exportId}` in `backend/tests/contract/audit/audit.contract.spec.ts`.
- [ ] T008 [P] Add failing integration test asserting linear status transitions, optimistic concurrency, and history append in `backend/tests/integration/accounts-status.integration.spec.ts`.
- [ ] T009 [P] Add failing integration test validating audit export queueing, Service Bus dispatch, and 12-month/10 MB guardrails in `backend/tests/integration/audit-exports.integration.spec.ts`.
- [ ] T010 [P] Add failing React Testing Library integration test for dashboard filtering/search UX in `frontend/tests/integration/accounts-dashboard.spec.tsx`.
- [ ] T011 [P] Add failing Playwright end-to-end scenario for audit export request-to-download journey with notification checks in `frontend/tests/e2e/audit-export.spec.ts`.

## Phase 3: Backend Core Implementation
- [ ] T012 [P] Implement Account domain model with zod validation and retention metadata in `backend/src/modules/accounts/domain/account.model.ts`.
- [ ] T013 [P] Implement AccountHistory domain model capturing change feed metadata in `backend/src/modules/accounts/domain/account-history.model.ts`.
- [ ] T014 [P] Implement AccountStatusCatalog domain model enforcing linear transitions in `backend/src/modules/accounts/domain/account-status-catalog.model.ts`.
- [ ] T015 [P] Implement AuditExportRequest domain model including size tracking in `backend/src/modules/audit/domain/audit-export-request.model.ts`.
- [ ] T016 [P] Implement UserRole reference model with permission sets in `backend/src/modules/auth/domain/user-role.model.ts`.
- [ ] T017 [P] Implement NotificationSubscription domain model for follow-up and export alerts in `backend/src/modules/notifications/domain/notification-subscription.model.ts`.
- [ ] T018 Configure Cosmos DB module with Managed Identity authentication, container registration, and RU throttling policies in `backend/src/infrastructure/cosmos/cosmos.module.ts`.
- [ ] T019 [P] Implement Account repository with filtered queries and soft-delete awareness in `backend/src/modules/accounts/repositories/account.repository.ts`.
- [ ] T020 [P] Implement AccountHistory repository with timeline pagination in `backend/src/modules/accounts/repositories/account-history.repository.ts`.
- [ ] T021 [P] Implement AuditExportRequest repository handling queued jobs in `backend/src/modules/audit/repositories/audit-export.repository.ts`.
- [ ] T022 [P] Implement NotificationSubscription repository for subscription CRUD in `backend/src/modules/notifications/repositories/notification-subscription.repository.ts`.
- [ ] T023 Define account DTOs and mappers (list, detail, status transition) in `backend/src/modules/accounts/dtos/account.dto.ts`.
- [ ] T024 [P] Define account history DTOs and response shapes in `backend/src/modules/accounts/dtos/account-history.dto.ts`.
- [ ] T025 [P] Define audit export request/response DTOs in `backend/src/modules/audit/dtos/audit-export.dto.ts`.
- [ ] T026 Implement AccountService orchestrating filters, detail retrieval, and transition rules in `backend/src/modules/accounts/services/account.service.ts`.
- [ ] T027 [P] Implement AccountHistoryService supplying paginated timelines in `backend/src/modules/accounts/services/account-history.service.ts`.
- [ ] T028 [P] Implement AccountStatusService validating allowed transitions, approvals, and concurrency in `backend/src/modules/accounts/services/account-status.service.ts`.
- [ ] T029 [P] Implement AuditExportService managing validation, queue dispatch, and blob link generation in `backend/src/modules/audit/services/audit-export.service.ts`.
- [ ] T030 [P] Implement SearchService integrating Azure Cognitive Search in `backend/src/modules/search/services/account-search.service.ts`.
- [ ] T031 Implement GET `/accounts` endpoint in `backend/src/modules/accounts/controllers/accounts.controller.ts` mapping to AccountService list.
- [ ] T032 Implement GET `/accounts/{accountId}` endpoint in `backend/src/modules/accounts/controllers/accounts.controller.ts` including RBAC guard.
- [ ] T033 Implement GET `/accounts/search` endpoint in `backend/src/modules/accounts/controllers/accounts.controller.ts` returning combined results.
- [ ] T034 Implement POST `/accounts/{accountId}/status` endpoint in `backend/src/modules/accounts/controllers/account-status.controller.ts` with approval workflow hooks.
- [ ] T035 Implement GET `/accounts/{accountId}/history` endpoint in `backend/src/modules/accounts/controllers/account-history.controller.ts` with pagination params.
- [ ] T036 Implement POST `/accounts/{accountId}/audit-exports` endpoint in `backend/src/modules/audit/controllers/audit-exports.controller.ts` triggering background job.
- [ ] T037 Implement GET `/accounts/{accountId}/audit-exports/{exportId}` endpoint in `backend/src/modules/audit/controllers/audit-exports.controller.ts` returning status/download URLs.
- [ ] T038 Wire Azure Functions HTTP triggers to NestJS application in `backend/src/main.function.ts` and register modules in `backend/src/app.module.ts` for Static Web Apps deployment.

## Phase 4: Frontend Experience
- [ ] T039 Implement Azure AD AuthProvider with MSAL and protected routing in `frontend/src/app/providers/AuthProvider.tsx` and `frontend/src/app/routing/ProtectedRoute.tsx`.
- [ ] T040 Create HTTP client with token injection, error normalization, and retry hints in `frontend/src/lib/http/azureClient.ts`.
- [ ] T041 [P] Implement accounts API client methods in `frontend/src/lib/api/accountsClient.ts` mirroring backend contracts.
- [ ] T042 [P] Implement audit exports API client methods in `frontend/src/lib/api/auditExportsClient.ts`.
- [ ] T043 [P] Create React Query hooks for account list and search in `frontend/src/features/accounts/api/useAccounts.ts`.
- [ ] T044 [P] Create React Query hook for account detail and history in `frontend/src/features/accounts/api/useAccountDetail.ts`.
- [ ] T045 [P] Create React Query mutations for status transitions and audit export requests in `frontend/src/features/accounts/api/useAccountActions.ts`.
- [ ] T046 Implement filter panel component with Chakra controls in `frontend/src/features/accounts/components/AccountFilters.tsx`.
- [ ] T047 Implement account table/grid component with responsive columns in `frontend/src/features/accounts/components/AccountTable.tsx`.
- [ ] T048 Implement dashboard page assembling filters, table, and summary KPIs in `frontend/src/features/accounts/pages/AccountsDashboardPage.tsx`.
- [ ] T049 Implement account summary panel with owner/contact highlights in `frontend/src/features/accounts/components/AccountSummaryPanel.tsx`.
- [ ] T050 Implement history timeline component with grouped entries and accessibility labels in `frontend/src/features/accounts/components/HistoryTimeline.tsx`.
- [ ] T051 Implement status update drawer with validation and comment capture in `frontend/src/features/accounts/components/StatusUpdateDrawer.tsx`.
- [ ] T052 Implement audit export panel with range picker and progress state in `frontend/src/features/accounts/components/AuditExportPanel.tsx`.
- [ ] T053 Implement account detail page wiring summary, timeline, status updates, and export panel in `frontend/src/features/accounts/pages/AccountDetailPage.tsx`.
- [ ] T054 Implement global search page with tabbed accounts/history results in `frontend/src/features/accounts/pages/GlobalSearchPage.tsx`.
- [ ] T055 Wire feature routes, layout shell, and chakra theme tokens in `frontend/src/app/routing/index.tsx` and `frontend/src/app/theme/index.ts`.
- [ ] T056 [P] Implement notification provider observing export completion signals in `frontend/src/app/providers/NotificationProvider.tsx` and toast hooks.

## Phase 5: Integration & Operations
- [ ] T057 Configure Azure Service Bus messaging module and queue publisher in `backend/src/infrastructure/messaging/service-bus.module.ts` with retry/backoff policies.
- [ ] T058 Implement audit export processor function consuming queue messages, generating CSV/PDF, and updating status in `backend/src/functions/audit-export-processor/index.ts`.
- [ ] T059 Implement Cosmos change feed function emitting overdue follow-up notifications in `backend/src/functions/account-change-feed/index.ts`.
- [ ] T060 Integrate Application Insights telemetry, distributed tracing, and structured logging in `backend/src/infrastructure/telemetry/telemetry.module.ts` and middleware.
- [ ] T061 Implement Azure AD role guards and policy decorators in `backend/src/modules/auth/guards/roles.guard.ts` and `backend/src/modules/auth/decorators/roles.decorator.ts`.
- [ ] T062 Update Bicep templates for Cosmos containers, Service Bus, Blob exports, and Cognitive Search in `infrastructure/bicep/` modules with parameters for environments.
- [ ] T063 Update GitHub Actions workflow `infrastructure/github-actions/ci-cd.yml` to run lint, tests, build artifacts, and deploy Static Web Apps + Functions.
- [ ] T064 [P] Implement frontend export notification channel (polling or SignalR) in `frontend/src/features/notifications/useExportNotifications.ts` integrating with NotificationProvider.

## Phase 6: Optimization & Polish
- [ ] T065 [P] Add backend unit tests for AccountService, AuditExportService, and status validators in `backend/tests/unit/accounts.services.spec.ts` and `backend/tests/unit/audit-export.service.spec.ts`.
- [ ] T066 [P] Add frontend unit tests for filters, timeline, and export panel components in `frontend/tests/unit/accounts-components.spec.tsx`.
- [ ] T067 Add k6 smoke test script validating p95 budgets for dashboard and exports in `tests/perf/audit-export-smoke.js`.
- [ ] T068 [P] Add automated accessibility regression using axe in Playwright at `frontend/tests/accessibility/accounts.a11y.spec.ts`.
- [ ] T069 Update developer docs with run steps, test matrix, and troubleshooting in `README.md` and `specs/001-build-a-sales/quickstart.md`.
- [ ] T070 Document validation evidence (lint, typecheck, tests, k6, accessibility) in `docs/validation-report.md`.
- [ ] T071 Publish operations runbook covering alerts, purges, and escalation paths in `docs/runbook.md`.

## Dependencies
- Complete Phase 1 (T001–T005) before starting tests or implementation.
- Tests T006–T011 must be in place and failing before any implementation tasks they cover (T012 onward).
- Domain models T012–T017 must be finished before repositories and services (T019–T030).
- Services T026–T030 must exist before controller endpoints (T031–T037) and Azure Functions wiring (T038).
- Frontend API clients and hooks (T039–T045) must precede UI composition tasks (T046–T055).
- Integration tasks in Phase 5 depend on backend services/controllers and relevant frontend hooks.
- Polish tasks (Phase 6) run only after all functional work is complete.

## Parallel Execution Examples
- After T005, you can run tasks `task run T006`, `task run T007`, and `task run T010` in parallel to cover backend/ frontend contract specs.
- Once T017 completes, run `task run T019`, `task run T020`, and `task run T021` together to build Cosmos repositories.
- After T045 succeeds, execute `task run T046`, `task run T047`, and `task run T056` concurrently for independent frontend components and providers.
- When Phase 5 kicks off, `task run T057` and `task run T062` can proceed in parallel since messaging code and Bicep updates touch different areas.
