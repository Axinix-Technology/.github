# Engineering Standards

> **Document Reference:** AXN-DOCS-06-ENG-001  
> **Last Updated:** 2026-11-11  
> **Source:** Verified from repository source code, corporate portfolio, and `.agents/Rules/` files  
> **Authored By:** Axinix Technologies Documentation System

---

## 1. Architecture Conventions

### 1.1 The Populate-First Mandate

**Rule:** Every API capability that can be handled via `@register_model` + `services/<model>.py` + the Axinix Core populate pipeline **MUST** use the populate pipeline. Creating dedicated controllers/routers is **STRICTLY PROHIBITED** unless:

1. The capability physically cannot be handled via `populateHelper` (e.g., binary stream downloads, auth handshakes, external webhook signature verification)
2. The alternative would cause a verified, critical performance bottleneck

**Legitimate exceptions:** Authentication endpoints (`/auth/login`, `/auth/logout`, `/auth/me`), binary downloads (export PDFs, compressed archives), external webhook endpoints

### 1.2 Domain Logic in Services

All business logic belongs in `services/<model>.py` via lifecycle hooks:
- `before_create(context, data)` — Pre-creation validation and transformation
- `after_create(context, instance)` — Post-commit side effects (notifications, alerts)
- `before_update(context, data)` — Pre-update validation
- `after_update(context, instance)` — Post-update side effects
- `before_delete(context, instance)` — Pre-delete validation
- `after_delete(context, instance)` — Post-delete cleanup
- `before_read(context)` — Read-time scope enforcement
- `after_read(context, results)` — Read-time response transformation
- `get_policy_scope(context)` — Dynamic queryset scoping (ownership, tenant, capability-based)
- `check_permission(context)` — Tier 1 permission override

### 1.3 No String Role Checks

**Prohibited:**
```python
# NEVER DO THIS
if request.user.role.name == "Manager":
    ...
```

**Correct:**
```python
# Always use capability-based checks
from populate_core.policy.engine import PolicyEngine
if PolicyEngine.has_capability(context, "hr.read.all"):
    return queryset.all()
return queryset.filter(assigned_to=context.user)
```

---

## 2. Security Standards

### 2.1 Mandatory Audit Logging

**Core populate pipeline** (`/api/populate/`): Automatically audited at Stage 10 by `AuditLogger`. No developer action needed.

**Custom routes** (outside populate): MUST be registered under `/api/` and processed through `core.middleware.ApiAuditLogMiddleware`. Must also explicitly invoke `core.audit.helpers.record_audit_log(...)` for granular business events.

**Credential sanitization:** Always sanitize payloads via `core.audit.helpers.sanitize_payload(...)`. Never log raw passwords, auth tokens, secrets, or financial credentials.

### 2.2 Password Security

- Passwords are hashed via `django.contrib.auth.hashers.make_password` (PBKDF2) in `before_create`/`before_update` service hooks
- Password field is globally registered in `get_forbidden_fields` — never exposed in read responses
- Input passwords are capped at 128 characters (Axinix Core Stage 01 guard)

### 2.3 Multi-Tenant Isolation

- All queries must route through `TenantDatabaseRouter`
- Never manually `.filter(tenant_id=...)` — this is handled at the database routing layer
- Tenant context is propagated via Python `ContextVar` — thread-safe by design

### 2.4 Privilege Escalation Prevention

Mutations on sensitive fields (`is_superuser`, `is_staff`, `role`, `status`, `is_superadmin`) must check:
```python
if not context.is_superadmin:
    raise PopulateEngineError("ADMIN_REQUIRED", status=403)
```

---

## 3. Database Standards

### 3.1 Minimal Lock Windows

**Rule:** Never execute network I/O or heavy non-DB compute inside `transaction.atomic()` or while holding row locks (`select_for_update`). Keep DB locks under 10ms.

**Correct pattern for side effects after commit:**
```python
def after_create(self, context, instance):
    # Defer external calls to after transaction commits
    from django.db import transaction
    transaction.on_commit(lambda: self.send_notification(instance))
```

### 3.2 Composite Indexes for Soft-Delete

Never rely on a single-column index on `status`. Always create compound indexes pairing `status` with query keys:
```python
class Meta:
    indexes = [
        models.Index(fields=["status", "tenant_id", "created_at"]),
    ]
```

### 3.3 Prefetch Memory Safeguards

- Maximum populate depth: **3 levels** (enforced by `MAX_PREFETCH_DEPTH: 3`)
- Maximum child pagination: **50 items** (enforced by `CHILD_PAGINATION_CEILING: 50`)

### 3.4 Query Timeout

All MySQL queries run under a `SET SESSION max_execution_time = 5000` (5 seconds). This is enforced by the Axinix Core query compiler. Never attempt to disable this.

---

## 4. API Contract Standards

### 4.1 Standard Response Envelope

All successful populate responses follow this structure:
```json
{
    "success": true,
    "data": [...],
    "count": 120,
    "message": "Model created successfully.",
    "metadata": {
        "page": 1,
        "limit": 20,
        "total_pages": 6
    }
}
```

### 4.2 Correlation IDs

Every request must carry an `X-Request-ID`. The populate pipeline auto-generates one if absent and returns it in the response header. Custom routes must propagate this correlation ID.

### 4.3 Version Management

Workhub version is managed via `version.json` and synchronized with:
- `npm run version:bump:patch/minor/major`
- `npm run version:sync`

Mobile version must be manually synchronized in `pubspec.yaml`.

---

## 5. Frontend Standards

### 5.1 File Routing

Only valid page components belong in `src/pages/`. Modals, sub-components, and configuration files must be placed in `src/components/` or `src/constants/`.

### 5.2 Production Console Silencing

All `console.log`, `console.debug`, `console.info` are suppressed in production via `src/utils/logger.js`. Never bypass this by using raw console calls in feature code.

### 5.3 Chunk Size Limits

No vendor chunk may exceed 600 kB. City/geography data must use modular subpath imports — never bundle full geography JSON datasets.

---

## 6. Known Limitations and Bug Recipes

> See also: `.agents/Rules/known_limitations_and_recipes.md`

| Recipe | Rule |
|---|---|
| **Lock Windows** | Never network I/O inside `transaction.atomic()`. Max 10ms lock window. |
| **Composite Indexes** | Always `INDEX(status, tenant_id, created_at)` — never single-column `status` index. |
| **Prefetch Safety** | Max depth 3, max children 50 — enforced by Core config. |
| **Pipeline Caching** | Cache parsed DSL plans and authorization policies in-memory for hot paths. |
| **Image Safety** | 25MP max pixel ceiling via `Pillow.MAX_IMAGE_PIXELS`. |
| **Password Safety** | 128-character ceiling before bcrypt/PBKDF2. |
| **Linguistic Decisions** | Never use string checks (`endswith`, `startswith`, stopword lists). Route through `CognitiveLinguisticFrame`. |

---

## Related Documents

- [../02-shared-technology/axinix-core/ARCHITECTURE.md](../02-shared-technology/axinix-core/ARCHITECTURE.md)
- [../09-architecture-decisions/](../09-architecture-decisions/)
- [../00-discovery/DEPENDENCY-INVENTORY.md](../00-discovery/DEPENDENCY-INVENTORY.md)
