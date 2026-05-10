# Strategic Glue vNext — PRD Suite (Greenfield)

This folder contains a **greenfield** PRD set for building a new Strategic Glue platform from scratch.

## Documents

1. `01_Product_PRD.md`
   - Product vision, user personas, UX expectations, business rules, and acceptance criteria.
2. `02_System_Architecture.md`
   - Azure architecture, security, networking, hosting, observability, and environment design.
3. `03_Data_Model_Postgres.md`
   - Complete relational schema, DDL, indexes, constraints, retention, and migration strategy.
4. `04_API_Contracts_v1.md`
   - HTTP API v1 endpoints, request/response schemas, error conventions, and versioning policy.
5. `05_Delivery_Plan_and_NFRs.md`
   - NFRs, CI/CD, test strategy, launch plan, operations, and risk management.

## Key Positioning

- This PRD suite intentionally assumes **no dependency on legacy runtime logic**.
- UI should preserve the existing visual language and interaction feel, but all implementation details are rewritten for a fresh stack and cloud-native deployment.
