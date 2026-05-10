# 04 — API Contracts v1 (Greenfield)

Base path: `/api/v1`
Authentication: Bearer JWT unless endpoint is marked public.

## 1. Error Contract

```json
{
  "code": "string",
  "message": "string",
  "correlationId": "uuid"
}
```

## 2. Auth Endpoints

### POST /auth/login
Request:
```json
{ "email": "user@example.com", "password": "string" }
```
Response:
```json
{
  "accessToken": "jwt",
  "expiresInSeconds": 3600,
  "refreshToken": "opaque",
  "user": { "id": "uuid", "email": "user@example.com", "roles": ["admin"] }
}
```

### POST /auth/refresh
Request: `{ "refreshToken": "opaque" }`
Response: same as login.

### POST /auth/logout
Request: `{ "refreshToken": "opaque" }`
Response: `204`.

## 3. Clients

### GET /clients
- Admin only.
- Returns paged list.

### POST /clients
Create organization and optionally seed first client user.

## 4. Users

### POST /users
- Admin only.
- Creates user with role assignment.

### PATCH /users/{id}/status
- Admin only.
- Activate/deactivate user.

## 5. Documents

### POST /documents/upload
Multipart upload endpoint:
- `organizationId`
- `file`
- optional tags

Returns metadata record including storage reference.

### GET /documents
Query params: `organizationId`, `status`, `page`, `pageSize`.

### DELETE /documents/{id}
Soft delete record and mark blob for retention workflow.

## 6. Audit Runs

### POST /audit-runs
Request:
```json
{ "organizationId": "uuid", "documentIds": ["uuid"], "initiatedByUserId": "uuid" }
```
Response:
```json
{ "id": "uuid", "status": "pending" }
```

### GET /audit-runs/{id}
Returns run summary, category states, and timestamps.

### GET /audit-runs/{id}/categories/{categoryCode}
Returns category details, scores, outputs, and warnings.

## 7. Reviewer Actions

### POST /reviewer/actions
Request:
```json
{
  "categoryRunId": "uuid",
  "actionType": "approve|edit_score|rerun|escalate",
  "newScore": 7.4,
  "reason": "string"
}
```

## 8. Published Reports

### POST /published-audits
- Admin only.
- Publishes immutable version for an audit run.

### GET /published-audits/{organizationSlug}/latest
- Admin/client within tenant scope.

### GET /published-audits/{organizationSlug}/versions/{versionNo}
- Retrieve historical version.

## 9. Contract Governance

- OpenAPI spec generated and committed.
- Contract tests required for all endpoints above.
- Breaking changes require `v2` namespace.
