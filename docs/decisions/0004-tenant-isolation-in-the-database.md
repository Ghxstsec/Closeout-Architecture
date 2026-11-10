# ADR 0004: Tenant isolation is enforced by the database

## Context

The usual way to separate tenants in a SaaS application is to add `where firm_id = ?` to every query. That works until someone forgets it once. In Closeout, one forgotten filter means one pentest firm reading another firm's clients' open vulnerabilities, which is close to the worst possible incident.

## Decision

Tenant isolation is enforced by PostgreSQL Row-Level Security on every table that holds tenant data:

- The API connects with a role that is subject to RLS policies, and sets the current tenant at the start of each transaction.
- With no tenant set, policies return no rows. **The system fails closed.**
- Records belonging to another tenant return *not found*.
- The one legitimate cross-tenant read (the regression sweep) goes through a narrow database function that returns only identifiers. The real work then happens inside each tenant's own context.

Application-level checks stay in place too; RLS is the guarantee underneath them.

## Consequences

- A missing filter in application code returns nothing instead of leaking data.
- Every query has to run inside a transaction with the tenant set, which takes some discipline in the codebase and in tests.
- Background jobs carry the tenant with them, and each one sets it explicitly.
