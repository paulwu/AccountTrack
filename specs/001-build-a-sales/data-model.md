# Phase 1 Data Model – Sales Account Status Web Application

## Collections & Entities (Cosmos DB Core SQL API)

### Account
- **Partition Key**: `/accountId`
- **Fields**:
  - `id` (string, UUID) – unique identifier (mirrors `accountId`)
  - `accountId` (string) – business identifier, equals partition key
  - `name` (string, required, max 200 chars)
  - `ownerId` (string, required) – Azure AD object ID of assigned seller
  - `ownerName` (string)
  - `industry` (string, enum of controlled vocabulary)
  - `annualRecurringRevenue` (number, currency stored as minor units)
  - `region` (string, ISO 3166-2)
  - `status` (string, enum: Lead | Qualified | Proposal | Negotiation | ClosedWon | ClosedLost)
  - `statusSince` (datetime)
  - `lastTouch` (datetime)
  - `contacts` (array<ContactSummary>) – cached highlights for dashboard
  - `relationships` (array<object>) – optional links to opportunities/cases (id, type)
  - `retentionPolicy` (object) – `{ softDeleteAt?: datetime, purgeAt?: datetime }`
  - `createdAt` / `updatedAt` (datetime)
  - `createdBy` / `updatedBy` (string, Azure AD object IDs)
  - `_etag` (string, Cosmos concurrency token)
- **Indexes**:
  - Composite `(status ASC, region ASC)` for dashboard filters
  - Composite `(ownerId ASC, status ASC)` for manager reporting
  - Range index on `lastTouch` and `statusSince` for recency queries
- **Constraints**:
  - Status transitions validated server-side per linear pipeline
  - Soft deletion marks `retentionPolicy.softDeleteAt`; background job purges after 30 days post erasure approval

### AccountHistory
- **Partition Key**: `/accountId`
- **Fields**:
  - `id` (string, UUID)
  - `accountId` (string) – reference to parent account
  - `timestamp` (datetime, required)
  - `actorId` / `actorName` (string)
  - `actionType` (enum: StatusChanged, NoteAdded, MeetingLogged, AttachmentUploaded, ExportRequested, OwnershipChanged, Other)
  - `statusBefore` / `statusAfter` (string, optional depending on action)
  - `notes` (string, markdown), limited to 5,000 chars
  - `linkedOpportunityId` (string | null)
  - `metadata` (object) – e.g., meeting duration, attachment info
  - `source` (enum: UI, API, Automation)
  - `createdAt` (datetime)
- **Indexes**:
  - Range on `timestamp`
  - Composite `(actionType ASC, timestamp DESC)` for filtering
- **Change Feed**:
  - Projected to Azure Function for building materialized views and triggering notifications.

### AccountStatusCatalog
- **Partition Key**: `/status`
- **Fields**:
  - `status` (string) – primary key
  - `displayName` (string)
  - `description` (string)
  - `slaHours` (number)
  - `requiredFields` (array<string>) – e.g., proposal link before Negotiation
  - `allowedNextStatuses` (array<string>) – enforces linear pipeline (only next stage or closed states)
  - `requiresApproval` (boolean) – e.g., moving to ClosedWon
  - `createdAt` / `updatedAt`
- **Usage**: Reference data loaded at deploy time; cached in backend and frontend.

### AuditExportRequest
- **Partition Key**: `/accountId`
- **Fields**:
  - `id` (string, UUID)
  - `accountId` (string)
  - `requestedBy` (string, Azure AD object ID)
  - `requestedAt` (datetime)
  - `rangeStart` / `rangeEnd` (datetime)
  - `status` (enum: Pending, Processing, Completed, Failed)
  - `downloadUrls` (array<object>) – {type: "csv"|"pdf", url, expiresAt}
  - `fileSizeBytes` (number)
  - `errorDetails` (string | null)
- **Workflow**: Created synchronously; background function fulfills request if >10 MB.

### UserRole (Reference)
- **Partition Key**: `/role`
- **Fields**:
  - `role` (string: SalesRep | Manager | Admin)
  - `permissions` (array<string>) – capability strings synchronized with RBAC guard middleware
  - `description`

### NotificationSubscription
- **Partition Key**: `/accountId`
- **Fields**:
  - `id` (string)
  - `accountId` (string)
  - `subscriberId` (string, Azure AD)
  - `subscriptionType` (enum: OverdueFollowUp, StatusChange, ExportReady)
  - `channel` (enum: Email, Teams)
  - `createdAt`

## Relationships & Access Patterns
- Account → AccountHistory: 1-to-many, collocated by partition for efficient timeline queries.
- Account → AuditExportRequest: 1-to-many, ensures export lifecycle tied to account.
- AccountStatusCatalog provides validation metadata; cached server-side; rarely changes.
- NotificationSubscription enables targeted alerts for overdue follow-ups and export completion.

## Derived Projections
- Materialized views for dashboard metrics stored in Azure Table Storage or cached in Redis (optional) sourced from change feed.
- Search index (Azure Cognitive Search) built from Accounts + AccountHistory to power global search; pipeline defined in infrastructure docs.

## Data Governance
- Soft delete tracked via `retentionPolicy`; purge workflow triggered nightly and logged for compliance.
- PII fields (names, notes) flagged for encryption-at-rest; access audited via Application Insights events.
- GDPR erasure request logs stored in separate `compliance` container (managed outside scope but referenced by purge jobs).
