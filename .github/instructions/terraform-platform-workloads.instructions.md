---
applyTo: '**'
---

# Terraform Guidance for Platform Workloads Repositories

Use this guide in any repository that provisions Azure resources through Terraform and the shared `platform-workloads` automation.

---

## Core Workflow

- Register the workload in `platform-workloads/terraform/workloads/{portfolio}/{workload}.json` and set `configure_for_terraform: true` per environment.
- Run the platform-workloads pipeline to provision identities, role assignments, GitHub environments, and Terraform state storage.
- Store Terraform under `terraform/` with environment roots in `terraform/environments/{environment}` and shared modules in `terraform/modules`.
- Deploy through GitHub Actions using OIDC (no client secrets) and the environment-scoped federated credentials created by platform-workloads.

---

## What Platform Workloads Delivers

| Capability          | Description                                                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Service principals  | `spn-{workload}-{environment}` app registrations with federated credentials for GitHub Actions (`GitHub Actions deploying Azure resources`).           |
| Role assignments    | `Contributor` on `rg-{workload}-{env}-uksouth` plus optional roles supplied in the workload JSON.                                                      |
| GitHub environments | Secrets (`AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`, optional Sonar/NuGet) scoped to each environment.                              |
| Terraform state     | Storage account `sa{workload}{env}{region}tf` with container `{workload}-{environment}-tfstate` and data-access permissions for the service principal. |

Run the platform-workloads deployment whenever you add environments, change role requirements, or rotate credentials.

---

## Backend & Provider Templates

Keep backend references environment-specific and enable OIDC everywhere.

```hcl
terraform {
  required_version = ">= 1.6.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.0"
    }
  }

  backend "azurerm" {
    resource_group_name  = "rg-{workload}-{environment}-uksouth"
    storage_account_name = "sa{workload}{environment}ukstf"
    container_name       = "{workload}-{environment}-tfstate"
    key                  = "terraform.tfstate"
    use_oidc             = true
  }
}

provider "azurerm" {
  features {}
  use_oidc        = true
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
}

provider "azuread" {
  use_oidc  = true
  tenant_id = var.tenant_id
}
```

Keep `terraform/environments/{environment}/backend.tf` environment-specific. Avoid workspaces for prod/dev split; prefer separate directories with explicit backends.

---

## GitHub Actions Pattern

```yaml
name: Terraform Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: Production

    steps:
      - uses: actions/checkout@v4

      - name: Azure login (OIDC)
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Terraform init
        working-directory: terraform/environments/production
        run: terraform init
        env:
          ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
          ARM_USE_OIDC: true

      - name: Terraform plan
        working-directory: terraform/environments/production
        run: terraform plan -out=tfplan
        env:
          ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
          ARM_USE_OIDC: true

      - name: Terraform apply
        if: github.ref == 'refs/heads/main'
        working-directory: terraform/environments/production
        run: terraform apply -auto-approve tfplan
        env:
          ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
          ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
          ARM_USE_OIDC: true
```

Set `environment:` to the exact GitHub Environment created by platform-workloads (`Development`, `Production`, etc.). Keep `id-token: write` or OIDC will fail.

---

## Naming & Layout Conventions

- Resource groups: `rg-{workload}-{environment}-uksouth`.
- Region suffix only where needed (multi-region patterns or SKU limits).
- Character-limited resources (storage accounts, key vault) use lowercase and trimmed environment (`dev`, `prd`).
- Modules live in `terraform/modules/{module}` with `main.tf`, `variables.tf`, `outputs.tf`, and a short README.
- Environment roots compose modules and set environment-specific tags/values.

Tags should include at least `Environment`, `Workload`, `ManagedBy`, and `Repository` (GitHub slug).

---

## State Management

- State locking comes from Azure blob leases; no extra config needed.
- Mark sensitive outputs (`sensitive = true`) so values never leak to logs.
- Enable diagnostic logging on the state storage account through platform-workloads or Terraform to audit access.
- Separate state files by responsibility if destroy/apply order matters (for example `infrastructure.tfstate` vs `monitoring.tfstate`).

Example remote-state reference:

```hcl
data "terraform_remote_state" "infra" {
  backend = "azurerm"

  config = {
    resource_group_name  = "rg-{workload}-production-uksouth"
    storage_account_name = "sa{workload}produkstf"
    container_name       = "{workload}-production-tfstate"
    key                  = "infrastructure.tfstate"
    use_oidc             = true
  }
}
```

---

## Coding Guidelines

- Keep Terraform formatted (`terraform fmt -recursive`) and validated (`terraform validate`).
- Use module outputs instead of duplicating IDs or connection strings.
- Prefer data sources over hard-coded IDs when reading platform-workloads outputs (resource groups, service principals, key vaults).
- Guard critical resources with `lifecycle { prevent_destroy = true }` in production stacks.
- Document noteworthy infra changes in `terraform/CHANGELOG.md` or repository release notes.

---

## Reference Material

- `platform-workloads` README for automation behaviour and prerequisites.
- HashiCorp `azurerm` and `azuread` provider docs for resource schemas.
- GitHub guidance on configuring OIDC for Actions.

Copy this file into any workload repository that uses platform-workloads-managed Terraform to keep behaviour consistent.
