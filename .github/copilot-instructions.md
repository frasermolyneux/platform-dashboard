# GitHub Copilot Instructions - Platform Dashboard

This document provides context and guidelines for GitHub Copilot when working on the Platform Dashboard project.

---

## Project Overview

The Platform Dashboard is a development estate management portal built with:
- **Frontend**: React + TypeScript
- **Backend**: Azure Functions (Node.js)
- **Hosting**: Azure Static Web Apps
- **Authentication**: Azure AD Easy Auth
- **APIs**: GitHub REST API, GitHub App, Azure DevOps API
- **Data**: Azure Table Storage, Cosmos DB
- **Infrastructure**: Terraform
- **CI/CD**: GitHub Actions

**Purpose**: Provide visibility, governance, and compliance monitoring across all Molyneux.IO Azure Platform workloads.

**Custom Domains:**
- Production: `dashboard.molyneux.io`
- Development: `dashboard.dev.molyneux.io`

---

## Technology-Specific Guidelines

### React + TypeScript

**Code Style:**
```typescript
// Use functional components with TypeScript
interface WorkloadCardProps {
  workload: Workload;
  onRefresh: () => void;
}

export const WorkloadCard: React.FC<WorkloadCardProps> = ({ workload, onRefresh }) => {
  const [isLoading, setIsLoading] = useState(false);
  
  // Implementation
};
```

**Best Practices:**
- Always use TypeScript strict mode
- Define interfaces for all props and state
- Use React Hooks (useState, useEffect, useContext)
- Implement error boundaries for resilience
- Use React.memo() for expensive components
- Prefer composition over inheritance
- Keep components small and focused (<200 lines)

**State Management:**
- Use React Context for global state (auth, config)
- Use local state for component-specific data
- Consider TanStack Query (React Query) for API data fetching and caching

**Styling:**
- Use Tailwind CSS for styling
- Follow mobile-first responsive design
- Ensure accessibility (ARIA labels, keyboard navigation)

### Azure Functions (Node.js)

**Function Pattern:**
```typescript
import { app, HttpRequest, HttpResponseInit, InvocationContext } from '@azure/functions';

export async function getScanResults(
  request: HttpRequest,
  context: InvocationContext
): Promise<HttpResponseInit> {
  context.log('Processing governance scan request');
  
  try {
    // Implementation
    return { 
      status: 200, 
      jsonBody: { data } 
    };
  } catch (error) {
    context.log.error('Error:', error);
    return { status: 500, jsonBody: { error: error.message } };
  }
}

app.http('getScanResults', {
  methods: ['GET'],
  authLevel: 'function',
  handler: getScanResults
});
```
    FunctionContext executionContext)
{
    var logger = executionContext.GetLogger("ScanRepositoryGovernance");
    
    try 
    {
        // Implementation
        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(result);
        return response;
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "Error scanning repository");
        return req.CreateResponse(HttpStatusCode.InternalServerError);
    }
}
```

**Best Practices:**
- Use dependency injection for testability
- Implement proper error handling and logging
- Set appropriate timeout values
- Use async/await consistently
- Validate input parameters
- Return meaningful HTTP status codes

### GitHub App Authentication

**JWT Token Generation:**
```typescript
import jwt from 'jsonwebtoken';
import { Octokit } from '@octokit/rest';

async function getInstallationToken(installationId: number): Promise<string> {
  // Create JWT for GitHub App authentication
  const appId = process.env.GITHUB_APP_ID;
  const privateKey = process.env.GITHUB_APP_PRIVATE_KEY;
  
  const now = Math.floor(Date.now() / 1000);
  const payload = {
    iat: now,
    exp: now + 600, // 10 minutes
    iss: appId
  };
  
  const token = jwt.sign(payload, privateKey, { algorithm: 'RS256' });
  
  // Exchange JWT for installation token
  const octokit = new Octokit({ auth: token });
  const { data } = await octokit.apps.createInstallationAccessToken({
    installation_id: installationId
  });
  
  return data.token;
}
```

**Best Practices:**
- Cache installation tokens (valid for 1 hour)
- Implement token refresh logic
- Use fine-grained permissions
- Handle rate limiting (5,000 requests/hour)
- Implement retry logic with exponential backoff

### Azure Static Web Apps

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
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/api/*"]
  },
  "responseOverrides": {
    "401": {
      "redirect": "/.auth/login/aad",
      "statusCode": 302
    }
  }
}
```

**Authentication:**
```typescript
// Access user info from Easy Auth
interface UserInfo {
  userId: string;
  userDetails: string;
  identityProvider: string;
  userRoles: string[];
}

async function getCurrentUser(): Promise<UserInfo> {
  const response = await fetch('/.auth/me');
  const payload = await response.json();
  return payload.clientPrincipal;
}
```

**Best Practices:**
- Use Easy Auth for authentication (no custom auth code)
- Configure route-based authorization
- Set appropriate CORS policies
- Use environment variables for configuration
- Implement API proxying for backend calls

### Data Access Patterns

**Azure Table Storage:**
```typescript
import { TableClient } from '@azure/data-tables';

const tableClient = TableClient.fromConnectionString(
  process.env.AZURE_STORAGE_CONNECTION_STRING,
  'GovernanceScans'
);

// Upsert entity
await tableClient.upsertEntity({
  partitionKey: 'platform-dashboard',
  rowKey: scanId,
  timestamp: new Date(),
  complianceScore: 85,
  status: 'compliant'
});

// Query entities
const entities = tableClient.listEntities({
  queryOptions: { 
    filter: "PartitionKey eq 'platform-dashboard'" 
  }
});
```

**Cosmos DB:**
```typescript
import { CosmosClient } from '@azure/cosmos';

const client = new CosmosClient(process.env.COSMOS_CONNECTION_STRING);
const container = client.database('dashboard').container('scans');

// Create document
await container.items.create({
  id: scanId,
  workload: 'platform-monitoring',
  timestamp: new Date().toISOString(),
  results: { /* ... */ }
});

// Query with SQL
const { resources } = await container.items
  .query({
    query: "SELECT * FROM c WHERE c.workload = @workload ORDER BY c.timestamp DESC",
    parameters: [{ name: "@workload", value: "platform-monitoring" }]
  })
  .fetchAll();
```

**Best Practices:**
- Use Table Storage for high-volume, simple queries
- Use Cosmos DB for complex querying and time-series data
- Implement retry logic for transient failures
- Use connection pooling
- Set appropriate TTL for cached data

### Terraform Infrastructure

**Example Structure:**
```hcl
variable "environment" {
  description = "The environment name (e.g., dev, prd)"
  type        = string
}

variable "location" {
  description = "The location for resources"
  type        = string
  default     = "uksouth"
}

# Static Web App
resource "azurerm_static_web_app" "dashboard" {
  name                = "swa-platform-dashboard-${var.environment}-${var.location}"
  location            = var.location
  resource_group_name = azurerm_resource_group.dashboard.name
  sku_tier            = var.environment == "prd" ? "Standard" : "Free"
  sku_size            = var.environment == "prd" ? "Standard" : "Free"
}

# Custom domain
resource "azurerm_static_web_app_custom_domain" "dashboard" {
  static_web_app_id = azurerm_static_web_app.dashboard.id
  domain_name       = var.environment == "prd" ? "dashboard.molyneux.io" : "dashboard.dev.molyneux.io"
  validation_type   = "cname-delegation"
}
```

**Best Practices:**
- Use variables for environment-specific values
- Use consistent naming conventions
- Tag all resources appropriately
- Use managed identities where possible
- Document variables with descriptions
- Use terraform modules for reusability

---

## API Integration Guidelines

### GitHub REST API

**Rate Limiting:**
```typescript
async function checkRateLimit(octokit: Octokit): Promise<void> {
  const { data } = await octokit.rateLimit.get();
  const remaining = data.resources.core.remaining;
  
  if (remaining < 100) {
    const resetTime = new Date(data.resources.core.reset * 1000);
    console.warn(`Low rate limit: ${remaining} remaining. Resets at ${resetTime}`);
  }
}
```

**Pagination:**
```typescript
import { Octokit } from '@octokit/rest';

async function getAllRepositories(org: string): Promise<Repository[]> {
  const octokit = new Octokit({ auth: token });
  const repos: Repository[] = [];
  
  for await (const response of octokit.paginate.iterator(
    octokit.repos.listForOrg,
    { org, per_page: 100 }
  )) {
    repos.push(...response.data);
  }
  
  return repos;
}
```

### Error Handling

**Robust Error Handling:**
```typescript
class GovernanceError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public details?: any
  ) {
    super(message);
    this.name = 'GovernanceError';
  }
}

async function scanRepository(repo: string): Promise<ScanResult> {
  try {
    // Implementation
  } catch (error) {
    if (error.status === 404) {
      throw new GovernanceError('Repository not found', 404, { repo });
    } else if (error.status === 403) {
      throw new GovernanceError('Insufficient permissions', 403, { repo });
    }
    throw new GovernanceError('Scan failed', 500, { repo, error: error.message });
  }
}
```

---

## Testing Guidelines

**Unit Tests (Jest + React Testing Library):**
```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { WorkloadCard } from './WorkloadCard';

describe('WorkloadCard', () => {
  it('displays workload name', () => {
    const workload = { name: 'platform-dashboard', status: 'passing' };
    render(<WorkloadCard workload={workload} onRefresh={jest.fn()} />);
    
    expect(screen.getByText('platform-dashboard')).toBeInTheDocument();
  });
  
  it('calls onRefresh when button clicked', async () => {
    const onRefresh = jest.fn();
    render(<WorkloadCard workload={workload} onRefresh={onRefresh} />);
    
    await userEvent.click(screen.getByRole('button', { name: /refresh/i }));
    expect(onRefresh).toHaveBeenCalled();
  });
});
```

**Integration Tests:**
```typescript
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.get('/api/governance/scan', () => {
    return HttpResponse.json({ status: 'compliant', score: 95 });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## Security Best Practices

**Never commit secrets:**
- Use Azure Key Vault for sensitive data
- Use environment variables for configuration
- Use managed identities where possible

**Input Validation:**
```typescript
import { z } from 'zod';

const ScanRequestSchema = z.object({
  repository: z.string().min(1).max(100),
  owner: z.string().min(1).max(100),
  branch: z.string().optional().default('main')
});

export async function validateRequest(req: HttpRequest) {
  const body = await req.json();
  return ScanRequestSchema.parse(body); // Throws if invalid
}
```

**CORS Configuration:**
```typescript
// Only for local development - production uses SWA config
const corsHeaders = {
  'Access-Control-Allow-Origin': process.env.ALLOWED_ORIGIN || 'https://dashboard.molyneux.io',
  'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
  'Access-Control-Allow-Headers': 'Content-Type, Authorization'
};
```

---

## Performance Optimization

**Caching Strategy:**
```typescript
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  ttl: number;
}

class CacheService {
  private cache = new Map<string, CacheEntry<any>>();
  
  get<T>(key: string): T | null {
    const entry = this.cache.get(key);
    if (!entry) return null;
    
    const age = Date.now() - entry.timestamp;
    if (age > entry.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  set<T>(key: string, data: T, ttl: number = 3600000): void {
    this.cache.set(key, { data, timestamp: Date.now(), ttl });
  }
}
```

**Batch API Calls:**
```typescript
// Instead of sequential calls
const repos = await Promise.all(
  repoNames.map(name => octokit.repos.get({ owner, repo: name }))
);
```

---

## Code Organization

**Project Structure:**
```
src/
├── web/                    # React frontend
│   ├── components/        # Reusable components
│   ├── pages/             # Page components
│   ├── services/          # API clients
│   ├── hooks/             # Custom React hooks
│   ├── context/           # React Context providers
│   ├── types/             # TypeScript type definitions
│   └── utils/             # Utility functions
├── api/                    # Azure Functions
│   ├── governance/        # Governance scanning functions
│   ├── status/            # Status/metrics functions
│   ├── shared/            # Shared utilities
│   └── types/             # Shared type definitions
└── terraform/              # Terraform infrastructure
    ├── main.tf
    ├── variables.tf
    ├── modules/
    └── tfvars/
```

---

## Common Patterns

### Governance Scanning
Always follow this pattern when implementing governance rules:
1. Fetch data from API
2. Compare against defined rules
3. Generate violation list
4. Calculate compliance score
5. Store results in Cosmos DB
6. Return results to frontend

### API Response Format
Standardize all API responses:
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    message: string;
    code: string;
    details?: any;
  };
  metadata?: {
    timestamp: string;
    cached: boolean;
  };
}
```

---

## Accessibility Requirements

- All interactive elements must be keyboard accessible
- Use semantic HTML elements
- Provide ARIA labels for icon buttons
- Ensure color contrast meets WCAG AA standards
- Support screen readers
- Test with keyboard-only navigation

---

## Documentation Standards

**Code Comments:**
```typescript
/**
 * Scans a repository for governance compliance against defined rules.
 * 
 * @param owner - The repository owner (organization or user)
 * @param repo - The repository name
 * @param rules - The governance rules to validate against
 * @returns A promise resolving to the scan results
 * @throws {GovernanceError} If the scan fails or repository is inaccessible
 */
export async function scanRepository(
  owner: string,
  repo: string,
  rules: GovernanceRules
): Promise<ScanResult> {
  // Implementation
}
```

---

## Environment Variables

Always use these naming conventions:

**Frontend (.env):**
```
REACT_APP_API_URL=https://dashboard.molyneux.io/api
REACT_APP_ENVIRONMENT=production
```

**Backend (local.settings.json):**
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "GITHUB_APP_ID": "123456",
    "GITHUB_APP_PRIVATE_KEY": "-----BEGIN RSA PRIVATE KEY-----...",
    "COSMOS_CONNECTION_STRING": "AccountEndpoint=...",
    "TABLE_STORAGE_CONNECTION_STRING": "DefaultEndpointsProtocol=..."
  }
}
```

---

## When to Ask for Clarification

Request clarification when:
- Governance rule definitions are ambiguous
- API permissions are unclear
- Data schema needs to be extended
- New workload groups need to be added
- Performance requirements need definition
- Security implications are significant

---

## Related Documentation

- [Architecture](../docs/architecture.md)
- [Roadmap](../docs/roadmap.md)
- [Azure Static Web Apps Docs](https://learn.microsoft.com/en-us/azure/static-web-apps/)
- [GitHub Apps Documentation](https://docs.github.com/en/apps)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
