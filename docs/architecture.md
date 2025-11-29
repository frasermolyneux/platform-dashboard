# Platform Dashboard - Architecture

This document describes the technical architecture, design decisions, and implementation details of the Platform Dashboard.

---

## Table of Contents

- [Overview](#overview)
- [Architecture Diagram](#architecture-diagram)
- [Frontend Architecture](#frontend-architecture)
- [Backend Architecture](#backend-architecture)
- [Authentication & Authorization](#authentication--authorization)
- [Data Storage Strategy](#data-storage-strategy)
- [API Integration](#api-integration)
- [Infrastructure as Code](#infrastructure-as-code)
- [Security Architecture](#security-architecture)
- [Performance & Scalability](#performance--scalability)
- [Monitoring & Observability](#monitoring--observability)
- [Disaster Recovery](#disaster-recovery)

---

## Overview

The Platform Dashboard is built as a serverless application using Azure Static Web Apps, combining a React frontend with Azure Functions backend, authenticated via Azure AD Easy Auth.

**Key Design Principles:**
- **Serverless-First**: Minimize infrastructure management
- **Security by Default**: Authentication required for all access
- **Cost-Effective**: Pay only for usage, leverage free tiers
- **Developer-Friendly**: Modern tooling and familiar patterns
- **Resilient**: Graceful degradation and error handling

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Internet                                │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
              ┌──────────────────────────────┐
              │  Custom Domain               │
              │  dashboard.molyneux.io       │
              │  dashboard.dev.molyneux.io   │
              └──────────┬───────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────────┐
│              Azure Static Web App                              │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  Easy Auth (Azure AD)                    │ │
│  └──────────────────┬──────────────────┬────────────────────┘ │
│                     │                  │                      │
│        ┌────────────▼──────────┐  ┌───▼──────────────────┐   │
│        │   React Frontend      │  │  Azure Functions     │   │
│        │   (TypeScript)        │  │  (Node.js)           │   │
│        │                       │  │                      │   │
│        │  - Dashboard Views    │  │  - Governance Scans  │   │
│        │  - Governance UI      │  │  - GitHub API Calls  │   │
│        │  - Cost Charts        │  │  - Data Aggregation  │   │
│        │  - Status Cards       │  │  - Webhook Handler   │   │
│        └───────────────────────┘  └──────┬───────────────┘   │
└───────────────────────────────────────────┼───────────────────┘
                                            │
                    ┌───────────────────────┼───────────────────┐
                    │                       │                   │
                    ▼                       ▼                   ▼
         ┌──────────────────┐    ┌──────────────┐   ┌─────────────────┐
         │ Azure Table      │    │ Cosmos DB    │   │ Application     │
         │ Storage          │    │              │   │ Insights        │
         │                  │    │              │   │                 │
         │ - API Cache      │    │ - Governance │   │ - Telemetry     │
         │ - Recent Status  │    │   Results    │   │ - Metrics       │
         │ - Metrics        │    │ - History    │   │ - Logs          │
         └──────────────────┘    └──────────────┘   └─────────────────┘
                    │                       │
                    └───────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   Azure Key Vault     │
                    │                       │
                    │ - GitHub App Key      │
                    │ - API Secrets         │
                    │ - Connection Strings  │
                    └───────────────────────┘

                    External APIs:
         ┌─────────────────────────────────────┐
         │  GitHub REST API + GitHub App       │
         │  Azure Resource Graph API           │
         │  Azure Cost Management API          │
         │  Azure DevOps REST API (future)     │
         └─────────────────────────────────────┘
```

---

## Frontend Architecture

### Technology Stack
- **Framework**: React 18+
- **Language**: TypeScript 5+
- **Build Tool**: Vite
- **State Management**: React Context + TanStack Query
- **Styling**: Tailwind CSS
- **Charts**: Recharts
- **HTTP Client**: Fetch API with wrapper

### Component Structure

```
src/web/
├── components/
│   ├── common/                 # Shared components
│   │   ├── Button/
│   │   ├── Card/
│   │   ├── LoadingSpinner/
│   │   └── ErrorBoundary/
│   ├── dashboard/              # Dashboard-specific
│   │   ├── WorkloadCard/
│   │   ├── StatusIndicator/
│   │   └── MetricsSummary/
│   ├── governance/             # Governance components
│   │   ├── ComplianceScore/
│   │   ├── ViolationList/
│   │   └── RuleDetails/
│   └── layout/                 # Layout components
│       ├── Header/
│       ├── Navigation/
│       └── Footer/
├── pages/                      # Route components
│   ├── DashboardPage/
│   ├── GovernancePage/
│   ├── WorkloadDetailPage/
│   └── SettingsPage/
├── services/                   # API clients
│   ├── api.ts                  # Base API client
│   ├── governanceApi.ts
│   ├── githubApi.ts
│   └── costApi.ts
├── hooks/                      # Custom React hooks
│   ├── useAuth.ts
│   ├── useWorkloads.ts
│   └── useGovernanceScans.ts
├── context/                    # React Context
│   ├── AuthContext.tsx
│   ├── ConfigContext.tsx
│   └── ThemeContext.tsx
├── types/                      # TypeScript definitions
│   ├── workload.ts
│   ├── governance.ts
│   └── api.ts
├── utils/                      # Utility functions
│   ├── formatters.ts
│   ├── validators.ts
│   └── constants.ts
└── App.tsx                     # Root component
```

### State Management Strategy

**React Context** for:
- Authentication state
- User preferences
- Application configuration
- Theme settings

**TanStack Query** for:
- API data fetching
- Caching and invalidation
- Background refetching
- Optimistic updates

**Local State** for:
- UI state (modals, dropdowns)
- Form state
- Component-specific data

### Routing

```typescript
// React Router v6
const routes = [
  { path: '/', element: <DashboardPage /> },
  { path: '/governance', element: <GovernancePage /> },
  { path: '/workload/:name', element: <WorkloadDetailPage /> },
  { path: '/cost', element: <CostPage /> },
  { path: '/settings', element: <SettingsPage /> },
  { path: '*', element: <NotFoundPage /> }
];
```

---

## Backend Architecture

### Azure Functions API

**Runtime**: Node.js 20 LTS  
**Azure Functions Runtime**: v4  
**Package Manager**: npm

**Function Structure**:

```
src/api/
├── governance/
│   ├── scanRepository/         # POST - Scan single repo
│   ├── scanAllRepositories/    # POST - Batch scan
│   ├── getComplianceReport/    # GET - Retrieve results
│   └── getComplianceTrends/    # GET - Historical data
├── status/
│   ├── getWorkflowRuns/        # GET - GitHub Actions status
│   ├── getDeployments/         # GET - Deployment history
│   └── getPullRequests/        # GET - Open PRs
├── cost/
│   ├── getCostByWorkload/      # GET - Workload costs
│   └── getCostTrends/          # GET - Cost over time
├── github-webhook/             # POST - Webhook handler
├── shared/                     # Shared code
│   ├── github/
│   │   ├── client.ts           # GitHub API client
│   │   └── auth.ts             # GitHub App auth
│   ├── storage/
│   │   ├── tableStorage.ts     # Table Storage client
│   │   └── cosmosDb.ts         # Cosmos DB client
│   ├── cache/
│   │   └── cacheManager.ts     # Caching logic
│   └── utils/
│       ├── logger.ts
│       └── errorHandler.ts
└── types/                      # Shared TypeScript types
```

### Function Execution Model

**Triggers:**
- HTTP Triggers: All user-facing endpoints
- Timer Triggers: Scheduled scans (every 6 hours)
- Queue Triggers: Asynchronous processing (future)

**Authorization Levels:**
- `anonymous`: Webhook endpoints (with HMAC validation)
- `function`: All other endpoints (called from authenticated SWA frontend)

### Background Processing

**Timer Functions:**
```typescript
// Scheduled scan every 6 hours
import { app, Timer } from '@azure/functions';

export async function scheduledGovernanceScan(
  timer: Timer,
  context: InvocationContext
): Promise<void> {
  context.log('Starting scheduled governance scan');
  
  const workloads = await getWorkloadConfig();
  for (const workload of workloads) {
    await queueScan(workload);
  }
}

app.timer('scheduledGovernanceScan', {
  schedule: '0 0 */6 * * *', // Every 6 hours
  handler: scheduledGovernanceScan
});
```

---

## Authentication & Authorization

### Azure AD Easy Auth

**Authentication Flow:**

```
1. User navigates to dashboard.molyneux.io
2. Static Web App checks auth state
3. If not authenticated:
   - Redirect to /.auth/login/aad
   - Azure AD login prompt
   - Redirect back to application
4. SWA injects auth headers in API requests
5. Functions validate auth automatically
```

**Configuration (staticwebapp.config.json):**

```json
{
  "routes": [
    {
      "route": "/api/*",
      "allowedRoles": ["authenticated"]
    },
    {
      "route": "/*",
      "allowedRoles": ["authenticated"]
    }
  ],
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad",
      "statusCode": 302
    }
  },
  "globalHeaders": {
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Content-Security-Policy": "default-src 'self' https://*.molyneux.io"
  }
}
```

### User Context Access

**Frontend:**
```typescript
const response = await fetch('/.auth/me');
const { clientPrincipal } = await response.json();

interface ClientPrincipal {
  userId: string;
  userDetails: string; // email
  identityProvider: 'aad';
  userRoles: string[];
}
```

**Backend (Azure Functions):**
```typescript
const userId = req.headers['x-ms-client-principal-id'];
const userName = req.headers['x-ms-client-principal-name'];
```

### Future RBAC (Optional)

Define roles in Azure AD:
- **Admin**: Full access, can modify governance rules
- **Viewer**: Read-only access
- **Operator**: Can trigger scans, view reports

---

## Data Storage Strategy

### Azure Table Storage

**Use Cases:**
- API response caching (short-lived)
- Recent workflow run status
- Quick-access metrics

**Schema Example:**
```typescript
interface CacheEntity {
  partitionKey: string;    // 'github-api-cache'
  rowKey: string;          // Hash of request
  data: string;            // JSON stringified response
  timestamp: Date;
  ttl: number;             // Seconds until expiry
}
```

**Advantages:**
- Very low cost ($0.045/GB/month)
- Fast key-based lookups
- No schema enforcement (flexible)

### Azure Cosmos DB

**Use Cases:**
- Governance scan results (historical)
- Compliance score trends
- Violation tracking over time
- Complex querying needs

**Container Design:**

**Container: `governance-scans`**
```json
{
  "id": "scan-20250109-platform-monitoring",
  "workload": "platform-monitoring",
  "repository": "frasermolyneux/platform-monitoring",
  "scannedAt": "2025-01-09T10:30:00Z",
  "complianceScore": 85,
  "overallStatus": "partial",
  "checks": {
    "branchProtection": { /* ... */ },
    "requiredFiles": [ /* ... */ ]
  },
  "violations": [ /* ... */ ],
  "_ts": 1704794400
}
```

**Partition Key**: `workload` (good distribution, common query pattern)

**Container: `cost-history`**
```json
{
  "id": "cost-202501-platform-monitoring",
  "workload": "platform-monitoring",
  "month": "2025-01",
  "totalCost": 234.56,
  "dailyCosts": [ /* ... */ ],
  "_ts": 1704794400
}
```

**Indexing Policy:**
```json
{
  "indexingMode": "consistent",
  "includedPaths": [
    { "path": "/workload/*" },
    { "path": "/scannedAt/*" },
    { "path": "/complianceScore/*" }
  ],
  "excludedPaths": [
    { "path": "/checks/*" }
  ]
}
```

**Advantages:**
- Global distribution (if needed)
- Rich querying (SQL-like)
- Automatic indexing
- Time-series data support

### Caching Strategy

**Multi-Layer Cache:**

```
Request → In-Memory Cache (5 min)
              ↓ (miss)
          Table Storage Cache (1 hour)
              ↓ (miss)
          GitHub API / Azure API
              ↓
          Store in Table Storage
              ↓
          Return to client
```

---

## API Integration

### GitHub App Authentication

**Setup:**
1. Create GitHub App in organization
2. Generate private key
3. Install app with permissions:
   - Actions: Read
   - Contents: Read
   - Pull Requests: Read
   - Workflows: Read
   - Administration: Read (for branch protection)
   - Environments: Read
   - Code scanning alerts: Read

**Authentication Flow:**

```typescript
import jwt from 'jsonwebtoken';
import { Octokit } from '@octokit/rest';

class GitHubAppClient {
  private appId: string;
  private privateKey: string;
  private installationId: number;
  private tokenCache: { token: string; expiresAt: Date } | null = null;

  async getInstallationToken(): Promise<string> {
    // Check cache
    if (this.tokenCache && this.tokenCache.expiresAt > new Date()) {
      return this.tokenCache.token;
    }

    // Generate JWT
    const now = Math.floor(Date.now() / 1000);
    const payload = {
      iat: now,
      exp: now + 600, // 10 minutes
      iss: this.appId
    };
    const jwtToken = jwt.sign(payload, this.privateKey, { algorithm: 'RS256' });

    // Exchange for installation token
    const octokit = new Octokit({ auth: jwtToken });
    const { data } = await octokit.apps.createInstallationAccessToken({
      installation_id: this.installationId
    });

    // Cache token (valid for 1 hour)
    this.tokenCache = {
      token: data.token,
      expiresAt: new Date(Date.now() + 3600000)
    };

    return data.token;
  }

  async getClient(): Promise<Octokit> {
    const token = await this.getInstallationToken();
    return new Octokit({ auth: token });
  }
}
```

### Rate Limiting

**GitHub API Limits:**
- GitHub App: 5,000 requests/hour per installation
- REST API: Additional limits per endpoint

**Rate Limit Handling:**

```typescript
async function makeGitHubRequest<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  try {
    const result = await fn();
    
    // Check rate limit
    const octokit = await githubClient.getClient();
    const { data } = await octokit.rateLimit.get();
    
    if (data.resources.core.remaining < 100) {
      logger.warn('Low rate limit', { remaining: data.resources.core.remaining });
    }
    
    return result;
  } catch (error) {
    if (error.status === 403 && error.message.includes('rate limit')) {
      const resetTime = new Date(error.response.headers['x-ratelimit-reset'] * 1000);
      throw new Error(`Rate limit exceeded. Resets at ${resetTime}`);
    }
    
    if (retries > 0 && error.status >= 500) {
      await sleep(1000 * (4 - retries));
      return makeGitHubRequest(fn, retries - 1);
    }
    
    throw error;
  }
}
```

### Azure Cost Management API

**Authentication**: Managed Identity

```typescript
import { DefaultAzureCredential } from '@azure/identity';
import { CostManagementClient } from '@azure/arm-costmanagement';

const credential = new DefaultAzureCredential();
const client = new CostManagementClient(credential, subscriptionId);

// Query cost for last 30 days
const scope = `/subscriptions/${subscriptionId}`;
const query = {
  type: 'Usage',
  timeframe: 'MonthToDate',
  dataset: {
    granularity: 'Daily',
    aggregation: {
      totalCost: {
        name: 'Cost',
        function: 'Sum'
      }
    },
    grouping: [
      {
        type: 'Dimension',
        name: 'ResourceGroupName'
      }
    ]
  }
};

const result = await client.query.usage(scope, query);
```

---

## Infrastructure as Code

### Terraform Structure

```
terraform/
├── main.tf                     # Entry point
├── variables.tf                # Variable definitions
├── outputs.tf                  # Output values
├── providers.tf                # Provider configuration
├── tfvars/
│   ├── dev.tfvars             # Development variables
│   └── prd.tfvars             # Production variables
└── modules/
    ├── static-web-app/        # Static Web App + functions
    ├── storage/               # Table Storage
    ├── cosmos-db/             # Cosmos DB
    ├── key-vault/             # Key Vault
    ├── app-insights/          # Application Insights
    └── managed-identity/      # User-assigned identity
```

**main.tf:**
```hcl
terraform {
  required_version = ">= 1.14.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.54.0"
    }
  }
  
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}

module "static_web_app" {
  source = "./modules/static-web-app"
  
  name                = "swa-platform-dashboard-${var.environment}-${var.location}"
  location            = var.location
  resource_group_name = azurerm_resource_group.dashboard.name
  sku_tier            = var.environment == "prd" ? "Standard" : "Free"
  custom_domain       = var.environment == "prd" ? "dashboard.molyneux.io" : "dashboard.dev.molyneux.io"
}

module "cosmos_db" {
  source = "./modules/cosmos-db"
  
  account_name        = "cosmos-platform-dashboard-${var.environment}"
  location            = var.location
  resource_group_name = azurerm_resource_group.dashboard.name
  
  containers = [
    { name = "governance-scans", partition_key = "/workload" },
    { name = "cost-history", partition_key = "/workload" }
  ]
}

module "storage_account" {
  source = "./modules/storage"
  
  account_name        = "stpdashboard${var.environment}${random_string.unique.result}"
  location            = var.location
  resource_group_name = azurerm_resource_group.dashboard.name
}
```

---

## Security Architecture

### Defense in Depth

**Layer 1: Network**
- HTTPS only (TLS 1.2+)
- Azure Static Web Apps built-in DDoS protection
- Custom domain with SSL certificates

**Layer 2: Authentication**
- Azure AD authentication required
- No anonymous access
- Token-based API access

**Layer 3: Authorization**
- Role-based access (future)
- Least privilege principle

**Layer 4: Data**
- Encryption at rest (Azure Storage + Cosmos)
- Encryption in transit (HTTPS)
- Secrets in Key Vault only

**Layer 5: Application**
- Input validation
- Output encoding
- CSRF protection (SWA handles)
- Content Security Policy

### Secrets Management

**Never in Code:**
- GitHub App private key
- Storage connection strings
- Cosmos DB keys
- API tokens

**Key Vault Usage (Terraform):**
```hcl
resource "azurerm_key_vault" "dashboard" {
  name                = "kv-dashboard-${var.environment}"
  location            = var.location
  resource_group_name = azurerm_resource_group.dashboard.name
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"
  
  enable_rbac_authorization = true
}

resource "azurerm_key_vault_secret" "github_app_private_key" {
  name         = "github-app-private-key"
  value        = var.github_app_private_key
  key_vault_id = azurerm_key_vault.dashboard.id
}
```

**Function Access:**
```typescript
import { SecretClient } from '@azure/keyvault-secrets';
import { DefaultAzureCredential } from '@azure/identity';

const credential = new DefaultAzureCredential();
const client = new SecretClient(keyVaultUrl, credential);

const secret = await client.getSecret('github-app-private-key');
const privateKey = secret.value;
```

---

## Performance & Scalability

### Static Web App Scaling
- **CDN Edge Caching**: Global distribution
- **Auto-scaling**: Handled by Azure
- **Cold start**: Minimal (static assets cached)

### Azure Functions Scaling
- **Consumption Plan**: Auto-scale based on load
- **Max instances**: 200 (default)
- **Scale-out duration**: <30 seconds
- **Warm instances**: 1+ (Standard plan)

### Caching Strategy
- **Browser cache**: 1 hour for static assets
- **API cache (Table Storage)**: 1 hour
- **In-memory cache**: 5 minutes
- **Cosmos DB cache**: None (source of truth)

### Performance Targets
- **Page Load**: <2 seconds (initial)
- **API Response**: <500ms (p95)
- **Governance Scan**: <30 seconds per repository
- **Batch Scan**: <5 minutes (38 repositories)

---

## Monitoring & Observability

### Application Insights

**Telemetry:**
- Page views and load times
- API request durations
- Exception tracking
- Custom events (governance scans, etc.)
- Dependency calls (GitHub API, Cosmos DB)

**Custom Metrics:**
```typescript
import { TelemetryClient } from 'applicationinsights';

const client = new TelemetryClient(instrumentationKey);

// Track governance scan
client.trackEvent({
  name: 'GovernanceScanCompleted',
  properties: {
    workload: 'platform-monitoring',
    complianceScore: 85,
    duration: 12.5
  }
});

// Track metric
client.trackMetric({
  name: 'ComplianceScore',
  value: 85
});
```

**Dashboards:**
- Overall application health
- API performance metrics
- GitHub API rate limit usage
- Cost trends

### Alerting

**Azure Monitor Alerts:**
- High API latency (>2 seconds)
- Function failures (>5% error rate)
- GitHub rate limit low (<100 remaining)
- Cost anomalies (>20% increase)

---

## Disaster Recovery

### Backup Strategy

**Cosmos DB:**
- Automatic continuous backup (30 days)
- Point-in-time restore capability

**Table Storage:**
- Geo-redundant storage (GRS)
- No backup (ephemeral cache data)

**Code & Infrastructure:**
- Git repository (GitHub)
- Terraform state in Azure Storage

### Recovery Objectives

**RTO (Recovery Time Objective)**: 1 hour
**RPO (Recovery Point Objective)**: 15 minutes

### Recovery Procedures

1. **Static Web App failure**: Redeploy via GitHub Actions
2. **Cosmos DB corruption**: Point-in-time restore
3. **Region outage**: Manual terraform apply to new region (future: multi-region)
4. **Data loss**: Rescan all repositories (~5 minutes)

---

## Future Enhancements

- **Multi-region deployment** for global users
- **Redis cache** for improved performance
- **Azure Front Door** for global load balancing
- **Queue-based processing** for async workloads
- **Event Grid** for real-time updates
- **Azure AD B2C** for external user access (if needed)

---

## Related Documentation

- [Roadmap](./roadmap.md)
- [GitHub Apps Documentation](https://docs.github.com/en/apps)
- [Azure Static Web Apps](https://learn.microsoft.com/en-us/azure/static-web-apps/)
- [Azure Functions Best Practices](https://learn.microsoft.com/en-us/azure/azure-functions/functions-best-practices)
