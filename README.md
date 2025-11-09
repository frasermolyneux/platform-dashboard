# Platform Dashboard

| Stage                   | Status                                                                                                                                                                                                                                         |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| DevOps Secure Scanning  | [![DevOps Secure Scanning](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/devops-secure-scanning.yml/badge.svg)](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/devops-secure-scanning.yml)    |
| Feature Development     | [![Feature Development](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/feature-development.yml/badge.svg)](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/feature-development.yml)             |
| Pull Request Validation | [![Pull Request Validation](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/pull-request-validation.yml/badge.svg)](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/pull-request-validation.yml) |
| Release to Production   | [![Release to Production](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/release-to-production.yml/badge.svg)](https://github.com/frasermolyneux/platform-dashboard/actions/workflows/release-to-production.yml)       |

---

## Documentation

* [Architecture](/docs/architecture.md) - Technical architecture and design decisions
* [Roadmap](/docs/roadmap.md) - Feature development roadmap and implementation phases

---

## Overview

The Platform Dashboard is a comprehensive development estate management portal that provides visibility, governance, and compliance monitoring across all workloads in the Molyneux.IO Azure Platform.

**Key Capabilities:**

- **Status Dashboard**: Real-time build and deployment status across all repositories
- **Governance Enforcement**: Repository compliance monitoring against defined policies
- **Security Overview**: Consolidated security alerts, vulnerabilities, and Dependabot notifications
- **Cost Management**: Azure resource consumption tracking per workload
- **Deployment History**: Visual timeline of deployments with DORA metrics
- **Compliance Reporting**: Branch protection, required files, workflow validation

---

## Technology Stack

| Component           | Technology                      |
| ------------------- | ------------------------------- |
| **Frontend**        | React + TypeScript              |
| **Backend**         | Azure Functions (Node.js)       |
| **Hosting**         | Azure Static Web Apps           |
| **Authentication**  | Azure AD (Easy Auth)            |
| **Data Storage**    | Azure Table Storage + Cosmos DB |
| **API Integration** | GitHub App + GitHub REST API    |
| **Infrastructure**  | Terraform                       |
| **CI/CD**           | GitHub Actions                  |

---

## Features

See the [Roadmap](/docs/roadmap.md) for detailed feature planning and implementation phases.

**Key Capabilities:**
- Repository status and pipeline visualization
- Governance compliance monitoring
- Branch protection rule validation
- Security alerts overview
- Cost management and tracking
- Deployment history and metrics
- Infrastructure drift detection

---

## Architecture

The solution uses a serverless architecture with Azure Static Web Apps:

```
                         Internet
                            │
                            ▼
              ┌──────────────────────────┐
              │  Custom Domain           │
              │  dashboard.molyneux.io   │
              └──────────┬───────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────────────┐
│                 Azure Static Web App                       │
│  ┌────────────────────┐      ┌─────────────────────────┐  │
│  │  React Frontend    │──────│  Azure Functions API    │  │
│  │  (TypeScript)      │      │  (Node.js)              │  │
│  └────────────────────┘      └─────────────────────────┘  │
│           │                            │                   │
│           │                            │                   │
└───────────┼────────────────────────────┼───────────────────┘
            │                            │
            │                            ├──► Azure Table Storage
            │                            ├──► Azure Cosmos DB
            │                            ├──► GitHub REST API
            │                            └──► Azure DevOps API
            │
            └──► Azure AD (Easy Auth)
```

**Domains:**
- Production: `dashboard.molyneux.io`
- Development: `dashboard.dev.molyneux.io`

**Authentication Flow:**
- Users authenticate via Azure AD using Static Web Apps Easy Auth
- No custom authentication code required
- Frontend receives authentication context
- API functions validate tokens automatically

**Data Sources:**
- **GitHub App**: Repository data, workflows, deployments (5,000 req/hr rate limit)
- **GitHub REST API**: Governance queries, branch protection, security alerts
- **Azure DevOps REST API**: Pipeline status (stretch goal)
- **Azure Resource Graph**: Resource inventory and compliance

---

## Governance Rules

The dashboard enforces configurable governance policies across repositories:

**Branch Protection:**
- Require pull request reviews (configurable count)
- Dismiss stale reviews
- Require status checks before merge
- Prevent force pushes and deletions
- Require conversation resolution

**Repository Settings:**
- Vulnerability alerts enabled
- Delete branch on merge
- Auto-merge capability
- Issues/Wiki/Projects configuration

**Required Files:**
- README.md
- CONTRIBUTING.md
- SECURITY.md
- LICENSE

**Required Workflows:**
- devops-secure-scanning.yml
- dependabot-automerge.yml
- Feature development pipeline
- Pull request validation

---

## Workload Grouping

Repositories are organized into logical workload families:

- **Portal Suite** - Core portal applications and services
- **Platform Services** - Infrastructure and shared platform components
- **Geo-Location** - Location-based services
- **Infrastructure** - Landing zones, workloads, connectivity
- **Utilities** - Reusable actions, templates, Terraform modules

Configuration is managed via `workloads.config.json` with customizable governance profiles per group.

---

## Local Development

```bash
# Install dependencies
npm install

# Run frontend (http://localhost:3000)
npm run dev

# Run Azure Functions locally (http://localhost:7071)
cd api
func start

# Run tests
npm test

# Build for production
npm run build
```

---

## Deployment

Deployment is fully automated via GitHub Actions:

1. **Feature Development**: Deploys to development environment on push to `main`
2. **Pull Request Validation**: Validates builds and runs tests
3. **Release to Production**: Deploys to production on release creation

Infrastructure is managed with Terraform and deployed to:
- Development: `sub-visualstudio-enterprise` (dashboard.dev.molyneux.io)
- Production: `sub-platform-management` (dashboard.molyneux.io)

---

## Related Projects

* [frasermolyneux/platform-workloads](https://github.com/frasermolyneux/platform-workloads) - Workload identity and RBAC configuration
* [frasermolyneux/platform-landing-zones](https://github.com/frasermolyneux/platform-landing-zones) - Azure landing zone management
* [frasermolyneux/actions](https://github.com/frasermolyneux/actions) - Reusable GitHub Actions

---

## Contributing

Please read the [contributing](CONTRIBUTING.md) guidance; this is a learning and development project.

---

## Security

Please read the [security](SECURITY.md) guidance; I am always open to security feedback through email or opening an issue.
