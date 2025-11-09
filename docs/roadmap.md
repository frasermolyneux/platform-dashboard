# Platform Dashboard - Development Roadmap

This document outlines the phased development approach for the Platform Dashboard, prioritizing features based on value delivery and technical dependencies.

---

## Phase 1 - MVP (Weeks 1-3)

**Goal**: Deliver a functional dashboard showing repository status and basic governance

**Status**: 🔄 In Progress

### Features

#### 1.1 Core Infrastructure
- [ ] Azure Static Web App deployment (Terraform)
- [ ] GitHub Actions CI/CD pipelines
- [ ] Azure AD authentication (Easy Auth)
- [ ] Development and Production environments with custom domains
- [ ] Basic React app scaffolding with TypeScript
- [ ] Node.js Azure Functions setup

#### 1.2 Repository Status Dashboard
- [ ] Repository listing by workload group
- [ ] Latest GitHub Actions workflow status
- [ ] Workflow run history (last 10 runs)
- [ ] Pull request count and status
- [ ] Deployment status indicators
- [ ] Last deployment timestamp

#### 1.3 Basic Governance Compliance
- [ ] Repository settings validation
  - Vulnerability alerts enabled
  - Delete branch on merge
  - Auto-merge settings
- [ ] Required files check
  - README.md
  - CONTRIBUTING.md
  - SECURITY.md
  - LICENSE
- [ ] Branch protection validation (main branch)
  - Require pull request reviews
  - Require status checks
  - Dismiss stale reviews

#### 1.4 Workload Configuration
- [ ] `workloads.config.json` schema definition
- [ ] Workload grouping (Portal, Platform, Infrastructure, etc.)
- [ ] Repository metadata management
- [ ] Manual scan trigger

**Deliverables:**
- Working dashboard accessible at `https://dashboard.molyneux.io` (Production)
- Development environment at `https://dashboard.dev.molyneux.io`
- Authenticated access via Azure AD
- Visual representation of repository health
- Compliance report showing pass/fail status

**Timeline**: Week 1-3

---

## Phase 2 - High Value Additions (Weeks 4-6)

**Goal**: Add cost visibility, infrastructure health, and historical tracking

**Status**: 📋 Planned

### Features

#### 2.1 GitHub App Integration
- [ ] Create and configure GitHub App
- [ ] Implement JWT-based authentication
- [ ] Installation token management
- [ ] Replace PAT authentication with GitHub App
- [ ] Webhook endpoint for real-time updates

#### 2.2 Cost Management Dashboard
- [ ] Azure Cost Management API integration
- [ ] Cost by workload/repository view
- [ ] Monthly cost trends (last 6 months)
- [ ] Budget threshold indicators
- [ ] Cost anomaly detection
- [ ] Subscription-level rollup
- [ ] Export cost reports

#### 2.3 Terraform State Health
- [ ] Azure Storage backend connection
- [ ] List all Terraform state files
- [ ] State file last modified timestamps
- [ ] Resource count per state
- [ ] Lock status monitoring
- [ ] State version tracking
- [ ] Drift detection (basic)

#### 2.4 Deployment Timeline
- [ ] Visual timeline/Gantt chart
- [ ] Deployment frequency metrics
- [ ] Environment progression tracking (Dev → Prod)
- [ ] Deployment duration statistics
- [ ] Success/failure rates
- [ ] Rollback event tracking

#### 2.5 Workload Health Scores
- [ ] Composite health score calculation (0-100)
- [ ] Score components:
  - Build success rate (20%)
  - Governance compliance (25%)
  - Security alerts (20%)
  - Deployment frequency (15%)
  - Documentation completeness (10%)
  - Cost vs. budget (10%)
- [ ] Historical score tracking
- [ ] Trend visualization

#### 2.6 Data Persistence
- [ ] Azure Table Storage for caching
- [ ] Cosmos DB for governance results
- [ ] Scheduled background scanning
- [ ] API response caching strategy

**Deliverables:**
- Cost visibility across all workloads
- Infrastructure health monitoring
- Historical compliance tracking
- Automated data refresh (hourly)

**Timeline**: Week 4-6

---

## Phase 3 - Deep Insights (Weeks 7-10)

**Goal**: Advanced analytics, drift detection, and dependency management

**Status**: 📋 Planned

### Features

#### 3.1 Infrastructure Drift Detection
- [ ] Terraform plan comparison
- [ ] Azure Resource Graph queries
- [ ] Drift identification per resource
- [ ] Visual diff display
- [ ] Alert on significant drift
- [ ] Drift remediation tracking

#### 3.2 Service Dependency Mapping
- [ ] Parse Terraform remote state references
- [ ] Build dependency graph
- [ ] Visual dependency visualization
- [ ] Impact analysis ("what depends on this?")
- [ ] Shared resource usage tracking
- [ ] API integration mapping

#### 3.3 Environment Configuration Comparison
- [ ] Side-by-side environment comparison
- [ ] SKU/tier comparison (Dev vs. Prod)
- [ ] Feature flag differences
- [ ] Configuration drift detection
- [ ] Settings recommendation engine

#### 3.4 Secret & Certificate Management
- [ ] Azure Key Vault integration
- [ ] Certificate expiration tracking
- [ ] Service principal credential monitoring
- [ ] Secret rotation reminders (30/60/90 days)
- [ ] Federated credential status
- [ ] Secret usage audit

#### 3.5 Azure Policy Compliance
- [ ] Azure Policy state integration
- [ ] Policy compliance by subscription
- [ ] Non-compliant resource highlighting
- [ ] Remediation task tracking
- [ ] Custom policy definition support

#### 3.6 Advanced Governance
- [ ] Workflow requirement validation
- [ ] Required secrets verification
- [ ] CODEOWNERS file validation
- [ ] SonarCloud integration
- [ ] Code coverage thresholds
- [ ] License compliance scanning

**Deliverables:**
- Drift detection and alerting
- Dependency impact analysis
- Environment parity validation
- Secret lifecycle management
- Enhanced compliance reporting

**Timeline**: Week 7-10

---

## Phase 4 - Advanced Features (Weeks 11-14)

**Goal**: Team insights, automation, and executive dashboards

**Status**: 📋 Planned

### Features

#### 4.1 Team & Contribution Insights
- [ ] Contributor activity tracking
- [ ] Commit frequency per repository
- [ ] Pull request metrics (opened, merged, closed)
- [ ] Code review response time
- [ ] CODEOWNERS coverage analysis
- [ ] Team velocity trends

#### 4.2 Infrastructure Resource Inventory
- [ ] Cross-subscription resource catalog
- [ ] Searchable/filterable resource list
- [ ] Resource age tracking
- [ ] Orphaned resource detection
- [ ] Tag compliance verification
- [ ] Subscription quota monitoring

#### 4.3 API & Integration Testing
- [ ] API endpoint catalog
- [ ] Integration test result display
- [ ] API performance metrics
- [ ] Swagger/OpenAPI synchronization
- [ ] Endpoint health monitoring

#### 4.4 Notification & Alerting
- [ ] Microsoft Teams integration
- [ ] Custom alert rule builder
- [ ] Email digest (daily/weekly)
- [ ] Webhook support for external tools
- [ ] Alert acknowledgment tracking

#### 4.5 Mobile-Optimized Views
- [ ] Responsive design optimization
- [ ] Executive summary dashboard
- [ ] Red/Amber/Green status indicators
- [ ] Critical-only alert filtering
- [ ] Touch-friendly interface

#### 4.6 Auto-Remediation (Experimental)
- [ ] Auto-create PRs for missing files
- [ ] Standard branch protection application
- [ ] Bulk governance operations
- [ ] Automated Dependabot PR approval
- [ ] Workflow template application

**Deliverables:**
- Team productivity insights
- Complete infrastructure visibility
- Proactive alerting system
- Mobile executive dashboard
- Governance automation tools

**Timeline**: Week 11-14

---

## Stretch Goals (Future Phases)

**Status**: 📋 Backlog

### Azure DevOps Integration
- [ ] Azure Pipelines status integration
- [ ] Azure Boards work item tracking
- [ ] Azure Repos branch policy validation
- [ ] Unified GitHub + ADO view

### Advanced Analytics
- [ ] DORA metrics dashboard
- [ ] Deployment frequency trends
- [ ] Lead time for changes
- [ ] Mean time to recovery (MTTR)
- [ ] Change failure rate

### AI-Powered Insights
- [ ] Anomaly detection (cost, deployments)
- [ ] Predictive failure analysis
- [ ] Recommendation engine
- [ ] Natural language queries

### Reporting & Exports
- [ ] PDF report generation
- [ ] PowerBI integration
- [ ] Custom report builder
- [ ] Scheduled report delivery

---

## Success Metrics

### Phase 1 Success Criteria
- Dashboard accessible to authorized users at both environments
- All repositories scanned for governance
- <2 second page load time
- 95%+ governance rule accuracy

### Phase 2 Success Criteria
- Cost data visible within 24 hours of Azure charges
- Terraform state health updated hourly
- Health scores calculated for all workloads
- GitHub App rate limits not exceeded

### Phase 3 Success Criteria
- Drift detected within 1 hour of change
- Dependency graph covers 90%+ of relationships
- Secret expiration alerts 30 days in advance
- Environment configuration comparison accurate

### Phase 4 Success Criteria
- Mobile dashboard usable on all devices
- Auto-remediation success rate >80%
- Notification delivery <5 minutes
- Resource inventory updated daily

---

## Risk Management

| Risk                       | Impact | Mitigation                                    |
| -------------------------- | ------ | --------------------------------------------- |
| GitHub API rate limits     | High   | Use GitHub App (5k req/hr), implement caching |
| Azure Cost API latency     | Medium | Cache results, update every 6 hours           |
| Terraform state access     | High   | Service Principal with minimal permissions    |
| Complex dependency mapping | Medium | Start simple, iterate based on patterns       |
| Authentication complexity  | Low    | Use Easy Auth, no custom auth code            |

---

## Technology Decisions

### Why Azure Static Web Apps?
- Integrated authentication (Easy Auth)
- Built-in Azure Functions backend
- Automatic SSL/CDN
- Cost-effective for this use case
- GitHub Actions integration

### Why GitHub App vs. PAT?
- Better rate limits (5,000 vs. 5,000 per user)
- Fine-grained repository permissions
- Organization-level installation
- Modern authentication pattern
- Better security posture

### Why Cosmos DB + Table Storage?
- **Table Storage**: Fast, cheap caching for ephemeral data
- **Cosmos DB**: Rich querying for governance results, time-series data
- Flexible schema for evolving requirements

### Why React + TypeScript?
- Strong typing reduces bugs
- Excellent tooling and ecosystem
- Component reusability
- Modern development experience

---

## Dependencies & Prerequisites

### Phase 1
- Azure subscription: `sub-platform-management` (Production), `sub-visualstudio-enterprise` (Development)
- Azure AD tenant access for Easy Auth configuration
- GitHub App creation and installation
- DNS configuration for `dashboard.molyneux.io` and `dashboard.dev.molyneux.io`

### Phase 2
- Azure Cost Management API access
- Terraform state storage access (Service Principal)
- GitHub App webhook configuration

### Phase 3
- Azure Resource Graph access across subscriptions
- Azure Policy read permissions
- Key Vault access policies

### Phase 4
- Microsoft Teams webhook (for notifications)
- PowerBI workspace (for reporting)

---

## Review & Iteration

This roadmap will be reviewed at the end of each phase to:
- Assess feature value vs. implementation effort
- Adjust priorities based on user feedback
- Incorporate learnings from previous phases
- Re-evaluate stretch goals for inclusion

**Last Updated**: November 9, 2025  
**Next Review**: End of Phase 1
