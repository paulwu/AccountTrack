# Quickstart – Sales Account Status Web Application

## Prerequisites
- Azure subscription with Cosmos DB, Azure AD tenant access, and permission to create Static Web Apps & Function Apps
- Node.js 20 LTS, npm 10+
- Azure CLI (`az`), Azure Functions Core Tools, Static Web Apps CLI (`swa`)
- GitHub CLI (`gh`) for CI/CD bootstrap
- Access to organization Azure AD app registration permissions

## Environment Setup
1. **Clone & Install**
   ```bash
   git clone <repo-url>
   cd accounttrack
   npm install --workspaces
   ```

2. **Provision Dev Resources** (Bicep via Azure CLI)
   ```bash
   az deployment sub create \
     --location eastus \
     --template-file infrastructure/bicep/dev.bicep \
     --parameters environment=dev
   ```
   Outputs include Cosmos endpoint/key, Storage account, Service Bus, and Azure AD app registration IDs.

3. **Configure Environment Variables**
   - Copy `infra/.env.example` to `frontend/.env.local` and `backend/.env.local`.
   - Populate values:
     - `VITE_AZURE_AD_CLIENT_ID`, `VITE_TENANT_ID`
     - `COSMOS_ENDPOINT`, `COSMOS_KEY` (dev only) or Managed Identity config
     - `SERVICE_BUS_QUEUE`, `BLOB_EXPORT_CONTAINER`
     - `APPLICATIONINSIGHTS_CONNECTION_STRING`

4. **Initialize Cosmos Containers**
   ```bash
   npm run backend:migrate
   ```
   Seeds reference data for `AccountStatusCatalog` and creates required indexes.

## Running Locally
- **Start Backend (Azure Functions)**
  ```bash
  npm run backend:dev
  ```
- **Start Frontend (Vite + SWA emulator)**
  ```bash
  npm run frontend:dev
  ```
- Access the SPA at `http://localhost:4280`. Login uses Azure AD dev app; ensure redirect URI `http://localhost:4280/.auth/login/aad/callback` is configured.

## Testing
1. **Lint & Type Check**
   ```bash
   npm run lint
   npm run typecheck
   ```
2. **Unit & Integration Tests**
   ```bash
   npm run test:frontend
   npm run test:backend
   ```
3. **End-to-End (Playwright)**
   ```bash
   npm run test:e2e -- --project=chromium
   ```
4. **Load Smoke (k6)**
   ```bash
   npm run test:load
   ```

## Demo Workflow
1. Login as sales rep (Azure AD test user).
2. Filter dashboard by status "Negotiation".
3. Open account, review timeline, add note, and advance status to "ClosedWon".
4. Trigger audit export (30-day window) and verify background job completes (check notifications panel).
5. Switch to manager role, run global search, and confirm aggregated metrics tiles.

## Deployment
1. **Configure GitHub Secrets**: `AZURE_CREDENTIALS`, `COSMOS_CONNECTION_STRING`, `STATIC_WEB_APP_TOKEN`.
2. **Push to `main`** triggers GitHub Actions workflow `ci-cd.yml`:
   - Install & lint/test both workspaces
   - Build frontend + backend
   - Deploy via Bicep `infrastructure/bicep/prod.bicep`
   - Publish Static Web App and Function App
3. **Post-Deploy Validation**
   - Run `npm run smoke:postdeploy`
   - Verify Application Insights dashboards and Cosmos metrics for throttling/errors.

## Operational Checks
- Monitor Azure Monitor alerts for p95 latency, Cosmos RU consumption, audit export backlog.
- Weekly review of accessibility scan results in Playwright reports.
- Monthly GDPR purge job audit: confirm erasure requests closed within 30 days.
