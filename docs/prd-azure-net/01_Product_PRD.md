# 01 — Product Requirements Document (Greenfield)

- Version: 2.0
- Date: May 10, 2026
- Product: Strategic Glue vNext
- Build Strategy: Greenfield implementation (no code reuse assumption)

## 1. Product Vision

Strategic Glue vNext is a web platform that automates and operationalizes marketing maturity audits across six domains: Brand, Customer, Offering, Communications, Sales, and Management. The system must be reliable for consultants and clear for client stakeholders, providing consistent scoring, evidence-backed insights, recommendations, and repeatable audit histories.

The new product is built from scratch using .NET and Azure managed services. While implementation is greenfield, the UI must preserve the current brand feel and visual design patterns so users experience continuity.

## 2. Business Goals

1. Reduce manual effort in audit scoring and summary generation.
2. Increase consistency of audit outcomes via structured workflows.
3. Create a secure, auditable system of record for all audit runs.
4. Enable fast consultant delivery with low operational burden.
5. Establish a platform foundation for future external integrations.

## 3. Success Metrics

### 3.1 Product KPIs
- Time from document upload to first draft audit report.
- Percentage of audits requiring manual score edits.
- Average time to complete reviewer queue.
- Number of successful client deliveries per month.

### 3.2 MVP Operational Metrics
- API non-AI p95 latency under 500ms at expected load.
- Successful audit run completion rate > 95% excluding user-cancelled runs.
- Document processing success rate > 99% for supported file types.

## 4. Personas and Users

1. Platform Admin
   - Sets up client organizations and users.
   - Uploads documents and initiates runs.
   - Reviews and publishes audits.
2. Client User
   - Views published reports and supporting details for their organization.

## 5. Roles and Access

- Roles: `admin`, `client`.
- Admin access: full data access across assigned organizations.
- Client access: restricted to their organization only.
- Auth approach: custom credentials-based auth with JWT + refresh tokens.

## 6. Scope

### 6.1 In Scope (Big Bang)
- Web application preserving current look-and-feel.
- .NET backend API (`/api/v1`) for auth, clients, docs, runs, review, publish.
- Azure App Service single-host deployment for web + API.
- Azure Database for PostgreSQL Flexible Server for all relational data.
- Azure Blob Storage for files/artifacts.
- Azure AI integration required in dev/test/prod.
- Private endpoint networking from day one.

### 6.2 Out of Scope (MVP)
- Mobile-native applications.
- Self-serve signup and billing.
- Multi-region active-active.

## 7. UX and Visual Requirements

### 7.1 Visual Continuity
- Preserve overall page composition, typography scale, card/list patterns, score visualizations, status indicators, and color personality.
- Preserve user flow semantics and major route map.
- Allow incremental UX improvements if they do not alter identity.

### 7.2 Core Screens
1. Login screen
2. Dashboard / audit list
3. Client profile and document manager
4. Audit run detail + status timeline
5. Reviewer queue and action drawer
6. Published report viewer

### 7.3 Accessibility
- WCAG 2.1 AA target for keyboard, focus, semantics, and contrast.

## 8. Functional Requirements

### 8.1 Authentication
- Login with email + password.
- Access token + refresh token rotation.
- Password reset flow (email token-based).
- Logout invalidates refresh token family.

### 8.2 Organization and User Admin
- Create/update/deactivate organizations.
- Create users and assign roles.
- Activate/deactivate user accounts.

### 8.3 Document Ingestion
- Upload PDF/DOCX/TXT/CSV/PPTX.
- Store binaries in blob and metadata in Postgres.
- Compute checksum and basic extraction status.

### 8.4 Audit Lifecycle
- Create audit run for an organization.
- Execute six domain analyses in parallel.
- Evaluate quality/policy rules and flag where needed.
- Route flagged categories to reviewer queue.
- Allow reviewer actions: approve, edit score, rerun, escalate.
- Publish final report snapshot with immutable version.

### 8.5 Reporting
- View latest and historical published reports.
- Display category scores, evidence notes, readiness findings, and recommendations.

## 9. Business Rules

1. Exactly six categories per audit run.
2. Score range is 0.0 to 10.0 with 0.1 precision.
3. Rerun lockout after 3 reruns per category within 24h.
4. Only admins can publish.
5. Published versions are immutable.

## 10. Edge Cases

- Partial AI failures: category can fail independently, run remains recoverable.
- Missing documents: run creation allowed, but warnings surfaced.
- Stale review actions: optimistic concurrency on category status updates.

## 11. Acceptance Criteria

1. A new organization can be created and users assigned with proper role restrictions.
2. Documents upload successfully and are listed with metadata.
3. Audit run processes six categories and persists status transitions.
4. Reviewer can approve/edit/rerun/escalate with full audit trail.
5. Published report is retrievable and versioned.
6. Client users cannot access cross-organization data.
