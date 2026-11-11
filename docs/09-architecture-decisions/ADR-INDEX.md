# ADR-001: Populate Engine as Universal API Backend

> **Document Reference:** AXN-DOCS-09-ADR-001  
> **Decision ID:** ADR-001  
> **Title:** Single canonical `/api/populate/` endpoint replacing per-entity routers  
> **Date:** Pre-November 2026 (original architecture decision by Arunbharathi)  
> **Status:** ACCEPTED — In production  
> **Authored By:** Axinix Technologies Documentation System

---

## Context

Traditional Django REST API architectures require developers to create five separate files per entity:
1. `models.py` — Schema definition
2. `serializers.py` — Input/output serialization
3. `views.py` — Controller with GET/POST/PUT/DELETE
4. `urls.py` — Router path registration
5. `permissions.py` — Custom permissions class

For a platform with 50+ database models (like Workhub), this produces 250+ files, 350+ endpoints, and extreme maintenance overhead. Security, audit logging, and database performance become developer-memory-dependent rather than structurally guaranteed.

## Problem

As Axinix's SaaS platforms grew past 20 models, the traditional approach produced:
- **Security gaps:** Developers occasionally forgot to apply field-level restrictions or permission checks
- **N+1 query explosions:** Developers forgot `select_related()`/`prefetch_related()`
- **Inconsistent response envelopes:** Different endpoints returned `{data: ...}` vs `{results: ...}` vs `{rows: ...}`
- **Duplicate audit code:** Each endpoint required its own audit logging, often incomplete or inconsistent
- **High velocity cost:** Adding a new model took 2-4 hours instead of 5-10 minutes

## Decision

Implement the **Populate Engine** (`django-populate-core`) as a single, shared framework with:
1. **One canonical route:** `POST /api/populate/{action}/{model}/` handles all CRUD operations for all models
2. **Declarative model registration:** `@register_model("model_name")` decorator to register any Django model
3. **Service hook pattern:** `@register_service("model_name")` for domain-specific business logic
4. **10-stage pipeline:** Fixed sequential stages for all requests (Ingress → Validation → Sanitization → Authorization → Parsing → Planning → Compilation → Execution → Serialization → Finalization)

## Reasoning

The Populate Engine provides structural guarantees that individual controllers cannot:
- **Authorization is structural:** PolicyEngine in Stage 04 always runs — cannot be bypassed by forgetting a permission decorator
- **Field security is structural:** Forbidden field lists apply to ALL queries — a developer cannot accidentally expose `password` or `salary`
- **Audit logging is structural:** Stage 10 always logs — 100% coverage vs ~40% with manual implementations
- **N+1 prevention is structural:** QueryPlanner in Stage 06 auto-plans joins — developers cannot forget `select_related()`
- **Tenant isolation is structural:** TenantDatabaseRouter routes based on context — developers cannot accidentally cross tenant boundaries

## Consequences

**Positive:**
- Near-zero routing boilerplate for new models
- Consistent API contract across all entities
- Centralized security enforcement
- Universal audit coverage
- Automated N+1 prevention

**Negative/Trade-offs:**
- Requires developers to understand Axinix Core's conventions before contributing
- Some legitimately complex custom endpoints still require non-populate routes
- Pipeline adds ~2-5ms overhead per request (measured by PipelineObserver)

## Supporting Source References

- [`Axinix-Core/populate_core/pipeline/executor.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/pipeline/executor.py)
- [`Axinix-Core/README.md`](file:///d:/Axinix%20Projects/Axinix-Core/README.md)
- [`tracker-v2/backend/requirements.txt`](file:///d:/Axinix%20Projects/tracker-v2/backend/requirements.txt)

---

# ADR-002: Physical Database Per Tenant (Multi-Tenant Architecture)

> **Decision ID:** ADR-002  
> **Title:** Physical database-per-tenant isolation via TenantDatabaseRouter  
> **Status:** ACCEPTED — In production (Workhub)

## Context

Multi-tenant SaaS platforms need to isolate tenant data. Common approaches:
1. Single database, `tenant_id` column on every table (row-level isolation)
2. Single database, separate schema per tenant (schema-level isolation)
3. Physical database per tenant (database-level isolation)

## Decision

**Physical database per tenant** implemented via Django's database routing system (`TenantDatabaseRouter`) in Axinix Core.

**Naming convention:** Tenant database name = `tracker_<subdomain>` (e.g., `tracker_axinix`, `tracker_clientxyz`)

**Tenant resolution:** Control Plane's `UserLogin` table maps email addresses to tenant databases. Authentication resolves the tenant at login time and embeds the database alias in the JWT token.

**Thread-safe context propagation:** Python `ContextVar` (`current_tenant_db_var`, `current_tenant_id_var`) propagates the tenant context across async operations without thread collision.

## Reasoning

- **Strongest data isolation:** Physical separation prevents any software bug from leaking data across tenants
- **Regulatory compliance:** Easier to demonstrate data isolation for compliance/GDPR
- **Backup granularity:** Each tenant's data can be backed up and restored independently
- **Performance isolation:** Slow queries by one tenant don't degrade other tenants' database performance

## Consequences

**Positive:** Maximal security and isolation; compliance-friendly; independent backup/recovery per tenant

**Negative:** More complex provisioning (each new tenant requires a new database + migrations); higher operational overhead for DBA management; connection pool management per tenant

## Source References

- [`Axinix-Core/populate_core/pipeline/context.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/pipeline/context.py) — `ContextVar` tenant propagation
- [`tracker-v2/knowledge_brain/ControlPlane/MODULE_BRAIN.md`](file:///d:/Axinix%20Projects/tracker-v2/knowledge_brain/ControlPlane/MODULE_BRAIN.md)

---

# ADR-003: Django Backend Promotion — Retiring Legacy Node.js Backend (Workhub)

> **Decision ID:** ADR-003  
> **Title:** Promote Django+Axinix-Core as canonical backend for Workhub, retiring Node.js  
> **Date:** Approximately October 2026 (git commit: `3e5c349`)  
> **Status:** ACCEPTED — Implemented in v4.1.0

## Context

Workhub (`tracker-v2`) was originally built with a Node.js backend. The Django+Axinix-Core populate engine was developed in parallel as a superior architecture.

## Decision

At v4.0 (commit `3e5c349 feat(backend): promote Django MySQL populate engine to canonical backend, retire legacy Node backend, and bump to v4.1.0`), Django with Axinix Core was promoted as the sole canonical backend.

## Reasoning

The Node.js backend did not benefit from the Populate Engine's structural security guarantees, audit logging, RBAC/ABAC enforcement, or multi-tenant isolation that Axinix Core provides.

## Source References

- `git -C "d:\Axinix Projects\tracker-v2" log --oneline` commit `3e5c349`
- [`tracker-v2/backend/requirements.txt`](file:///d:/Axinix%20Projects/tracker-v2/backend/requirements.txt)

---

# ADR-004: SHA-256 Client-Side Password Pre-Hashing (E-Commerce)

> **Decision ID:** ADR-004  
> **Title:** Browser-side SHA-256 pre-hashing before transmitting login credentials  
> **Status:** ACCEPTED — In production (E-Commerce)

## Context

Standard web login transmits passwords over HTTPS in plaintext (relying on TLS for transport security). The E-commerce platform chose an additional layer.

## Decision

The login page (`login.jsx`) uses the browser-native `crypto.subtle.digest("SHA-256")` API to hash the password client-side before sending it. The server (`DualPasswordBackend`) accepts both SHA-256 pre-hashed passwords (from the React UI) and raw passwords (from Django Admin or API testing tools).

## Reasoning

- Prevents plaintext passwords from ever leaving the browser
- Provides protection even in cases of misconfigured TLS or logging middleware that captures request bodies
- DualPasswordBackend maintains backward compatibility with Django Admin access

## Consequences

**Positive:** Additional password protection layer; passwords never transmitted in plaintext

**Negative:** Requires consistent client-side hashing; DualPasswordBackend complexity; not a substitute for HTTPS

## Source References

- [`E-commerce/README.md`](file:///d:/Axinix%20Projects/E-commerce/README.md) — Authentication & Security section
- `E-commerce/backend/users/backends.py` — DualPasswordBackend implementation
