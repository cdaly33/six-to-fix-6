# 05 — Delivery Plan, NFRs, and Operations

## 1. Non-Functional Requirements

### 1.1 Performance
- Expected concurrency: up to 5 active users.
- Non-AI endpoints p95 under 500ms.
- Upload endpoint supports files up to agreed practical cap (default 50MB).

### 1.2 Reliability
- Successful audit execution >95% excluding user cancellation.
- Retry/backoff policies for transient AI and storage failures.

### 1.3 Security
- Private endpoints for DB, Storage, Key Vault, and AI where supported.
- No secrets in code; all secrets via Key Vault.
- JWT key rotation supported.

### 1.4 Compliance
- No PII-heavy scope; apply standard platform controls.
- Full action logging for admin and reviewer actions.

## 2. CI/CD (GitHub Actions + Bicep)

Pipeline jobs:
1. Build web + API
2. Run tests
3. Run schema migration checks
4. Build deploy package
5. Deploy Bicep infrastructure
6. Deploy application to App Service
7. Smoke test endpoints

## 3. Test Strategy

- Unit tests: domain rules, lockout logic, policy evaluation.
- Integration tests: API + Postgres + Blob stubs/live test account.
- Contract tests: endpoint payloads and error codes.
- E2E tests: login, upload, run, review, publish.

## 4. Big-Bang Delivery Phases

1. Foundation and IaC
2. Auth and tenant model
3. Document ingestion
4. Audit orchestration and reviewer workflows
5. Reporting and publish
6. Hardening and launch

## 5. Go-Live Checklist

- All critical paths pass in `test`.
- Private endpoint connectivity validated.
- Backup/PITR verified.
- Alert rules enabled.
- Rollback plan documented.

## 6. Risks and Mitigations

1. Network misconfiguration with private endpoints.
   - Mitigation: scripted connectivity validation in deployment pipeline.
2. AI rate or timeout issues.
   - Mitigation: queueing, retries, circuit breakers, and clear failure states.
3. Big-bang integration defects.
   - Mitigation: end-to-end test coverage and dry-run cutover rehearsal.

## 7. Post-MVP

- Evaluate .NET Aspire to standardize local orchestration and observability developer experience.
