# 02 — System Architecture (Azure + .NET Greenfield)

## 1. Architecture Principles

1. Cloud-managed services first.
2. Private network boundaries by default.
3. Stateless application tier.
4. Explicit versioned API contracts.
5. Observability and diagnosability built-in.

## 2. Runtime Topology

- Single Azure App Service hosts:
  - ASP.NET Core API
  - Static web assets
- Azure PostgreSQL Flexible Server for relational state.
- Azure Blob Storage for files and generated artifacts.
- Azure Key Vault for application secrets and signing keys.
- Azure AI endpoint(s) for category and council processing.

## 3. Environment Model

- `dev`, `test`, `prod`
- Isolated resources per environment.
- Config from App Settings + Key Vault references.

## 4. Networking

- VNet integration for App Service.
- Private endpoints:
  - PostgreSQL
  - Storage Account
  - Key Vault
  - AI endpoints where supported
- Public network disabled for data services when feasible.

## 5. Security Architecture

- Managed identity for App Service.
- Least-privilege RBAC on all resources.
- TLS 1.2+ everywhere.
- Encryption at rest by Azure defaults.
- JWT signing keys from Key Vault.

## 6. Application Layers

1. Web UI layer
2. API layer (auth, client, docs, orchestration, reporting)
3. Domain services (policy, workflow, publication)
4. Data access layer (repositories + unit of work)
5. Integrations (AI, storage)

## 7. Deployment

- IaC: Bicep for all Azure resources.
- CI/CD: GitHub Actions pipeline with environment promotions.

## 8. Observability

- Application Insights for traces and dependencies.
- Log Analytics workspace.
- Alerts:
  - 5xx threshold
  - AI failure bursts
  - DB connectivity errors

## 9. Reliability

- Health endpoints and dependency checks.
- Retry policies with circuit breakers for AI/storage calls.
- PostgreSQL PITR enabled.
- Blob soft delete/versioning enabled.
