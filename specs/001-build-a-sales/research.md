# Phase 0 Research – Sales Account Status Web Application

## Overview
This document captures the decisions and evidence gathered to satisfy constitutional gates and eliminate remaining ambiguities before design.

## Decisions

### Cosmos DB Provisioning & Partitioning
- **Decision**: Use Azure Cosmos DB (Core SQL API) with partition key `accountId`, autoscale throughput 1k–10k RU/s, multi-region replication (primary: East US, secondary: West US).
- **Rationale**: `accountId` evenly distributes workload across accounts, aligns with per-account access, and keeps history lookups co-located. Autoscale protects against peak campaign loads while limiting cost during off hours.
- **Alternatives Considered**:
  - `ownerId` partition: rejected due to potential hot partitions when large teams own the majority of accounts.
  - Composite `{region, status}`: rejected—status transitions would cause cross-partition writes; region skew risk.

### Azure AD Authentication Flow
- **Decision**: Employ interactive Azure AD login using OAuth 2.1 authorization code flow with PKCE, enforcing MFA via Conditional Access. MSAL.js handles SPA auth; backend validates access tokens with Managed Identity.
- **Rationale**: Internal users already provisioned in Azure AD; interactive sign-in maintains session context and honors organization-wide MFA policies.
- **Alternatives Considered**:
  - Passwordless (FIDO2) only: deferred—leave optional for org-wide rollout; still compatible with Conditional Access.
  - Device code flow: unsuitable for SPA user experience.

### Audit Trail Export Limits
- **Decision**: Allow user-selected ranges up to 12 months, capped at 10 MB per export. Larger requests trigger background job segmentation with notification when ready.
- **Rationale**: Meets compliance reporting needs without overwhelming bandwidth; 12 months aligns with sales audit cycles.
- **Alternatives Considered**:
  - 90-day limit: rejected by compliance stakeholders; not enough history.
  - Unlimited export: risks timeouts and data exfiltration.

### Data Retention & GDPR Erasure
- **Decision**: Retain account and history data for 5 years unless marked for soft delete; upon GDPR erasure request, purge PII within 30 days while keeping anonymized metrics.
- **Rationale**: Balances sales analytics requirements with GDPR storage limitation; 30-day SLA aligns with constitutional privacy expectations.
- **Alternatives Considered**:
  - 7-year retention: unnecessary for majority of contracts.
  - Immediate hard delete: conflicts with financial reconciliation.

### Throttling & Offline Behavior
- **Decision**: Implement retry with exponential backoff for Cosmos 429 responses (max 3 retries). If sustained throttling occurs, enqueue mutation to Azure Service Bus queue processed by dedicated function; UI surfaces "queued" status to user.
- **Rationale**: Prevents data loss during spikes while maintaining responsiveness and transparency.
- **Alternatives Considered**:
  - No queuing: risk of lost updates and poor UX.
  - Infinite retries: increases latency and cost.

### Observability & Alerting
- **Decision**: Use Application Insights for distributed tracing, Log Analytics for centralized queries, Azure Monitor alerts for p95 latency breaches, throttling spikes, auth failures, and export job backlogs.
- **Rationale**: Meets constitutional observability requirements and integrates with existing Azure stack.
- **Alternatives Considered**: Third-party APM tools deferred to avoid additional compliance review.

### Accessibility Tooling
- **Decision**: Integrate Storybook with axe accessibility checks, Playwright + axe-core for automated regression, manual keyboard-only smoke tests before release.
- **Rationale**: Ensures WCAG 2.1 AA coverage across components and flows.
- **Alternatives Considered**: Manual audits only—insufficient for regression prevention.

## Outstanding Questions
- None (all high-impact ambiguities resolved or documented).

## Next Steps
- Proceed to Phase 1 design artifacts using decisions above.
