# 03 — Data Model and PostgreSQL Schema (Greenfield)

## 1. Design Objectives

- Full relational model for transactional integrity.
- Clear tenant boundaries by organization.
- Immutability for published audit snapshots.
- Query patterns optimized for light concurrent usage.

## 2. Canonical Tables

### 2.1 Identity & Access
- organizations
- users
- roles
- user_roles
- refresh_tokens
- password_reset_tokens

### 2.2 Document Domain
- documents
- document_tags
- document_extractions

### 2.3 Audit Domain
- audit_runs
- audit_categories
- audit_category_runs
- audit_category_outputs
- review_actions
- rerun_lockouts
- council_decisions
- published_audits

### 2.4 Ops Domain
- run_metrics_daily
- alert_events
- app_audit_log

## 3. PostgreSQL DDL (Initial)

```sql
create extension if not exists pgcrypto;

create table organizations (
  id uuid primary key default gen_random_uuid(),
  slug text not null unique,
  name text not null,
  status text not null check (status in ('active','inactive')),
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table users (
  id uuid primary key default gen_random_uuid(),
  organization_id uuid references organizations(id),
  email citext not null unique,
  password_hash text not null,
  display_name text not null,
  status text not null check (status in ('active','invited','disabled')),
  last_login_at timestamptz,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table roles (
  id uuid primary key default gen_random_uuid(),
  name text not null unique check (name in ('admin','client'))
);

create table user_roles (
  user_id uuid not null references users(id) on delete cascade,
  role_id uuid not null references roles(id) on delete cascade,
  assigned_at timestamptz not null default now(),
  primary key (user_id, role_id)
);

create table refresh_tokens (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references users(id) on delete cascade,
  token_hash text not null,
  expires_at timestamptz not null,
  revoked_at timestamptz,
  created_at timestamptz not null default now(),
  replaced_by_token_id uuid references refresh_tokens(id)
);

create table password_reset_tokens (
  id uuid primary key default gen_random_uuid(),
  user_id uuid not null references users(id) on delete cascade,
  token_hash text not null,
  expires_at timestamptz not null,
  used_at timestamptz,
  created_at timestamptz not null default now()
);
```

```sql
create table documents (
  id uuid primary key default gen_random_uuid(),
  organization_id uuid not null references organizations(id),
  uploaded_by_user_id uuid not null references users(id),
  storage_container text not null,
  storage_blob_path text not null,
  file_name text not null,
  mime_type text not null,
  size_bytes bigint not null,
  sha256_checksum text not null,
  status text not null check (status in ('uploaded','processing','ready','deleted')),
  uploaded_at timestamptz not null default now(),
  deleted_at timestamptz
);

create table document_tags (
  id uuid primary key default gen_random_uuid(),
  document_id uuid not null references documents(id) on delete cascade,
  tag text not null
);

create table document_extractions (
  id uuid primary key default gen_random_uuid(),
  document_id uuid not null references documents(id) on delete cascade,
  extractor_version text not null,
  extraction_status text not null check (extraction_status in ('pending','running','succeeded','failed')),
  extracted_text text,
  metadata_json jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
```

```sql
create table audit_categories (
  id uuid primary key default gen_random_uuid(),
  code text not null unique,
  display_name text not null
);

create table audit_runs (
  id uuid primary key default gen_random_uuid(),
  organization_id uuid not null references organizations(id),
  initiated_by_user_id uuid not null references users(id),
  status text not null check (status in ('pending','running','awaiting_review','completed','failed')),
  started_at timestamptz,
  completed_at timestamptz,
  current_version_no int not null default 1,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create table audit_category_runs (
  id uuid primary key default gen_random_uuid(),
  audit_run_id uuid not null references audit_runs(id) on delete cascade,
  category_id uuid not null references audit_categories(id),
  status text not null check (status in ('pending','running','flagged','approved','rerun_requested','failed')),
  ai_score numeric(4,1) check (ai_score between 0 and 10),
  reviewer_score numeric(4,1) check (reviewer_score between 0 and 10),
  confidence numeric(4,2),
  evidence_strength text,
  warning_codes jsonb not null default '[]'::jsonb,
  started_at timestamptz,
  completed_at timestamptz,
  version_no int not null default 1,
  unique (audit_run_id, category_id, version_no)
);

create table audit_category_outputs (
  id uuid primary key default gen_random_uuid(),
  category_run_id uuid not null unique references audit_category_runs(id) on delete cascade,
  payload_json jsonb not null,
  schema_version text not null,
  model_name text not null,
  model_deployment text not null,
  prompt_version text not null,
  created_at timestamptz not null default now()
);

create table review_actions (
  id uuid primary key default gen_random_uuid(),
  category_run_id uuid not null references audit_category_runs(id) on delete cascade,
  reviewer_user_id uuid not null references users(id),
  action_type text not null check (action_type in ('approve','edit_score','rerun','escalate')),
  previous_score numeric(4,1),
  new_score numeric(4,1),
  reason text,
  created_at timestamptz not null default now()
);

create table rerun_lockouts (
  id uuid primary key default gen_random_uuid(),
  category_run_id uuid not null unique references audit_category_runs(id) on delete cascade,
  lockout_until timestamptz,
  rerun_count_24h int not null default 0,
  updated_at timestamptz not null default now()
);

create table council_decisions (
  id uuid primary key default gen_random_uuid(),
  category_run_id uuid not null references audit_category_runs(id) on delete cascade,
  advocate_output jsonb not null,
  skeptic_output jsonb not null,
  judge_output jsonb not null,
  final_decision jsonb not null,
  created_at timestamptz not null default now()
);

create table published_audits (
  id uuid primary key default gen_random_uuid(),
  audit_run_id uuid not null references audit_runs(id) on delete cascade,
  version_no int not null,
  published_by_user_id uuid not null references users(id),
  published_payload_json jsonb not null,
  schema_version text not null,
  published_at timestamptz not null default now(),
  unique (audit_run_id, version_no)
);
```

```sql
create table run_metrics_daily (
  id uuid primary key default gen_random_uuid(),
  metric_date date not null,
  total_runs int not null,
  completed_runs int not null,
  flagged_runs int not null,
  avg_duration_seconds numeric(10,2) not null,
  created_at timestamptz not null default now(),
  unique(metric_date)
);

create table alert_events (
  id uuid primary key default gen_random_uuid(),
  event_name text not null,
  severity text not null,
  payload_json jsonb not null,
  occurred_at timestamptz not null default now()
);

create table app_audit_log (
  id uuid primary key default gen_random_uuid(),
  actor_user_id uuid references users(id),
  action text not null,
  entity_type text not null,
  entity_id uuid,
  details_json jsonb not null default '{}'::jsonb,
  created_at timestamptz not null default now()
);
```

## 4. Indexes

```sql
create index idx_audit_runs_org_created on audit_runs(organization_id, created_at desc);
create index idx_acr_run_category on audit_category_runs(audit_run_id, category_id);
create index idx_docs_org_uploaded on documents(organization_id, uploaded_at desc);
create index idx_review_actions_category_created on review_actions(category_run_id, created_at desc);
create index idx_warning_codes_gin on audit_category_runs using gin(warning_codes);
create index idx_published_payload_gin on published_audits using gin(published_payload_json);
```

## 5. Data Retention

- Documents: soft-delete then hard-delete after policy threshold.
- Refresh tokens: retain revoked tokens for security audit window.
- App audit logs: retain minimum 365 days.

## 6. Migration Policy

- Forward-only migrations.
- All schema changes via versioned migration scripts.
- Automated migration validation in CI against ephemeral Postgres.
