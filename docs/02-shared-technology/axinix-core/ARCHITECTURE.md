# Axinix Core — Architecture Overview

> **Document Reference:** AXN-DOCS-02-CORE-001  
> **Last Updated:** 2026-11-11  
> **Repository:** `d:\Axinix Projects\Axinix-Core`  
> **Package:** `django-populate-core` v1.0.0  
> **Implementation Status:** PRODUCTION STABLE  
> **Authored By:** Axinix Technologies Documentation System

---

## 1. Overview

**Axinix Core** (`django-populate-core`) is a reusable Django application that serves as the unified backend execution kernel for every Axinix Technologies platform. It replaces the traditional pattern of one-router-per-controller with a **single canonical API endpoint** (`/api/populate/`) that processes every request through a fixed 10-stage pipeline.

Any Django project that installs `django-populate-core` instantly gains:
- Full CRUD operations for every registered model
- Role-based and attribute-based access control (RBAC/ABAC)
- Automated relation prefetching (no N+1 queries)
- Non-blocking audit logging
- Multi-tenant database isolation
- Redis-backed rate limiting
- Standardized JSON response envelopes

---

## 2. Package Information

| Field | Value |
|---|---|
| **Package Name** | `django-populate-core` |
| **Version** | `1.0.0` |
| **License** | MIT |
| **Python** | `>=3.11` |
| **Django** | `>=5.0` |
| **DRF** | `>=3.14.0` |
| **Source** | `d:\Axinix Projects\Axinix-Core` |
| **GitHub** | `github.com/Axinix-Technology/Axinix-Core` |
| **Authors** | `engineering@axinix.com` |
| **Test Suite** | 15/15 tests passing |

### Optional Dependencies

| Extra | Libraries | Purpose |
|---|---|---|
| `mysql` | `mysqlclient>=2.2.0` | MySQL database driver |
| `redis` | `redis>=5.0.0`, `django-redis>=5.4.0` | Cache and rate limiting |
| `imaging` | `pillow>=10.0.0` | Image processing + decompression bomb protection |
| `full` | All above + `openpyxl>=3.1.0` | Complete installation |

---

## 3. Actual Pipeline Architecture

> [!IMPORTANT]
> The actual pipeline implementation differs from the marketing summary in the README. This section documents the **verified implementation** from `populate_core/pipeline/executor.py` and `stages.py`.

The pipeline has **10 stages** executed in strict sequential order by `PopulatePipeline.execute()`:

```
HTTP Request (any method: GET/POST/PUT/PATCH/DELETE)
       │
       ▼
┌─────────────────────────────────────────────┐
│  PopulateAPIView (views.py)                 │
│  IsAuthenticated check (or AllowAny for     │
│  public models)                             │
└─────────────┬───────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────┐
│  PopulatePipeline.execute() — executor.py   │
│  (wraps all stages in PipelineObserver)     │
└─────────────┬───────────────────────────────┘
              │
    01. INGRESS     ─── IngressStage.execute()
              │         Parse request → RequestContext
              ▼
    02. VALIDATION  ─── ValidationStage.execute()
              │         Schema/shape check → RequestContext
              ▼
    03. SANITIZATION ── SanitizationStage.execute()
              │         Identifier/parameter safety → RequestContext
              ▼
    04. AUTHORIZATION ─ AuthorizationStage.execute()
              │         RBAC/ABAC via PolicyEngine → RequestContext
              ▼
    05. PARSING     ─── ParsingStage.execute()
              │         Developer DSL → AST
              ▼
    06. PLANNING    ─── PlanningStage.execute()
              │         Join & Prefetch planning → query_plan
              ▼
    07. COMPILATION ─── CompilationStage.execute()
              │         AST → Django QuerySet
              ▼
    08. EXECUTION   ─── ExecutionStage.execute()
              │         CRUD + Domain Service Hooks → raw_result
              ▼
    09. SERIALIZATION ─ SerializationStage.execute()
              │         Output security & shaping → serialized_result
              ▼
    10. FINALIZATION ── FinalizationStage.execute_success/error()
                        AuditLogger.log() + Response envelope
                        → HTTP Response (with X-Request-ID header)
```

### Supported Actions

The pipeline supports these `action` values (from `context.py` `Action` enum):

| Action | Type | Description |
|---|---|---|
| `read` | Read | Fetch records with filtering, sorting, pagination |
| `create` | Mutation | Create a single record |
| `update` | Mutation | Update a single record by ID |
| `delete` | Mutation | Soft-delete a record |
| `report` | Read | Aggregated report query |
| `statistics` | Read | Statistical aggregation |
| `aggregate` | Read | Custom aggregation |
| `export` | Read | Data export |
| `bulk-create` | Mutation | Bulk record creation |
| `bulk-update` | Mutation | Bulk record update |
| `bulk-upsert` | Mutation | Bulk create-or-update |
| `bulk-delete` | Mutation | Bulk soft-delete |

---

## 4. Stage-by-Stage Documentation

### Stage 01: Ingress & Context Creation

**Class:** `IngressStage` (`populate_core/pipeline/stages.py`)

**Purpose:** Parse the incoming HTTP request and build the unified `RequestContext` dataclass that is passed through all subsequent stages.

**Input:** Raw DRF `Request` + URL `action`, `model_name`, `object_id` parameters

**Processing:**
1. Normalize `action` (lowercase, strip, replace `_` → `-`) and `model_name` (lowercase, strip)
2. Parse body data from `request.data` or fallback to `request.body` JSON → `request.POST`
3. For **read actions** (`read`, `report`, etc.): read parameters from both body AND query params (combined dict)
4. For **mutation actions** (`create`, `update`, etc.): read filter params from query params only; body data is the mutation payload
5. Deserialize JSON strings in `filter` and `populate` params if passed as serialized strings
6. Normalize top-level search params (`search`, `q`, `query`) into the filters dict
7. Set `request._is_populate_request = True` to prevent duplicate audit logging
8. Extract `X-Request-ID` header (or generate a fresh UUID)
9. Extract client IP via `get_client_ip()` and User Agent from `HTTP_USER_AGENT`
10. Build and return `RequestContext` dataclass

**Output:** `RequestContext` containing: `action`, `model_name`, `object_id`, `body`, `filters`, `populate`, `fields`, `ordering`, `pagination`, `user`, `request_id`, `ip_address`, `user_agent`, `path`, `method`

---

### Stage 02: Validation

**Class:** `ValidationStage` → `RequestValidator.validate(context)` (`populate_core/validation/`)

**Purpose:** Schema and shape validation of the request parameters.

---

### Stage 03: Sanitization

**Class:** `SanitizationStage` → `SecuritySanitizer.sanitize(context)` (`populate_core/security/sanitizer.py`)

**Purpose:** Identifier and parameter safety checks. Prevents injection attacks, enforces field allowlists, strips forbidden fields.

---

### Stage 04: Authorization

**Class:** `AuthorizationStage` → `PolicyEngine.authorize(context)` (`populate_core/policy/engine.py`)

**Purpose:** RBAC/ABAC authorization. Implements the 2-tier permission ladder:

1. **Tier 1 — Business Service:** If the domain service's `check_permission()` returns `True` → ALLOW (bypass engine check)
2. **Tier 2 — Populate Engine:** Evaluate `AccessPolicy` database records for RBAC/ABAC rules

If both fail → **FAIL CLOSED** (raises `PopulateEngineError` with HTTP 401/403)

**Key Design:** No string role-name checks (e.g., no `role.name == "manager"`). Access is evaluated strictly through database-backed `AccessPolicy` capabilities.

---

### Stage 05: Parsing

**Class:** `ParsingStage` → `DeveloperDSLParser.parse(context)` (`populate_core/query/parser.py`)

**Purpose:** Translate the developer-facing JSON filter/populate DSL into an internal AST (Abstract Syntax Tree).

**Output:** AST stored in `context.metadata["query_ast"]`

---

### Stage 06: Planning

**Class:** `PlanningStage` → `QueryPlanner.plan(ast, context.model_cls)` (`populate_core/query/planner.py`)

**Purpose:** Analyze the AST to plan optimal database joins and relation prefetching. Automatically determines which fields need `select_related` vs `prefetch_related` to prevent N+1 queries.

**Constraints:**
- Max prefetch depth: 3 levels (configurable via `POPULATE_CONFIG['MAX_PREFETCH_DEPTH']`)
- Max child pagination: 50 items (configurable via `POPULATE_CONFIG['CHILD_PAGINATION_CEILING']`)

**Output:** `query_plan` stored in `context.metadata["query_plan"]`

---

### Stage 07: Compilation

**Class:** `CompilationStage` → `QueryCompiler.compile(plan, context)` (`populate_core/query/compiler.py`)

**Purpose:** Translate the query plan into an actual Django `QuerySet`.

**Output:** Compiled Django `QuerySet` stored in `context.metadata["compiled_queryset"]`

---

### Stage 08: Execution

**Class:** `ExecutionStage` (`populate_core/pipeline/stages.py`) → dispatches to `crud/` executors

**Purpose:** Execute the compiled QuerySet, calling the appropriate CRUD executor and domain service lifecycle hooks.

**Executor Routing:**

| Action | Executor |
|---|---|
| `read` | `ReadExecutor.execute(queryset, context)` |
| `create`, `bulk-create`, `bulk-upsert` | `CreateExecutor.execute(context)` |
| `update`, `bulk-update` | `UpdateExecutor.execute(queryset, context)` |
| `delete`, `bulk-delete` | `DeleteExecutor.execute(queryset, context)` |
| `report`, `statistics` | `ReportExecutor.execute(queryset, context)` |

**Domain Service Hooks** (invoked by executors via registered service):

| Hook | When Called |
|---|---|
| `check_permission(context)` | Before authorization (Tier 1 permission ladder) |
| `before_create(context, data)` | Before database insertion |
| `after_create(context, instance)` | After successful database insertion |
| `before_update(context, data)` | Before database update |
| `after_update(context, instance)` | After successful database update |
| `before_delete(context, instance)` | Before soft-delete |
| `after_delete(context, instance)` | After successful soft-delete |
| `before_read(context)` | Before query execution |
| `after_read(context, results)` | After query results returned |

---

### Stage 09: Serialization

**Class:** `SerializationStage` → `OutputSerializer.serialize_result(execution_result, context)` (`populate_core/serialization/output.py`)

**Purpose:** Apply output security (strip forbidden fields, apply field projections) and shape the result into the standard response format.

---

### Stage 10: Finalization

**Class:** `FinalizationStage` (`populate_core/pipeline/stages.py`)

**Purpose:** Audit logging + build the final HTTP response.

**On Success (`execute_success`):**
1. Call `AuditLogger.log(context, serialized_result)` — non-blocking audit record
2. Build response metadata (pagination: `page`, `limit`, `total_pages`)
3. Add `summary` and `daily_ledger` to metadata if present in result
4. Generate auto-message (`"Model created/updated/deleted successfully"`)
5. Build `success_response(data, count, message, metadata, status_code)`
6. Status 201 for `create`, 200 for all others
7. Set `X-Request-ID` response header

**On Error (`execute_error`):**
1. Call `AuditLogger.log(context, {"error": str(error), "count": 0})` if context available
2. Build `error_response(error)` 
3. Set `X-Request-ID` response header

**Standard Response Envelope:**
```json
{
    "success": true,
    "data": [...],
    "count": 120,
    "message": "...",
    "metadata": {
        "page": 1,
        "limit": 20,
        "total_pages": 6
    }
}
```

---

## 5. Request Context (`RequestContext`)

The `RequestContext` dataclass (from `context.py`) is the single data structure passed through all 10 stages:

```python
@dataclass
class RequestContext:
    action: str                    # Normalized action name (e.g., "read", "create")
    model_name: str                # Normalized model name (lowercase)
    object_id: Any | None          # Record ID for update/delete
    body: dict | list              # Mutation payload (create/update data)
    filters: dict                  # Read filters (AST-compatible)
    populate: dict | list          # Relation population specification
    fields: list[str] | None       # Field projections
    ordering: list[str]            # Sort ordering (["-created_at", "name"])
    pagination: dict               # {"page": 1, "limit": 20, "offset": 0}
    user: Any                      # Django user object
    request_id: str                # UUID correlation ID (from X-Request-ID or generated)
    ip_address: str | None         # Client IP address
    user_agent: str | None         # User agent string
    path: str | None               # Request path
    method: str | None             # HTTP method
    tenant_context: dict | None    # Tenant metadata
    tenant_id: str | None          # Resolved tenant ID
    tenant_db: str                 # Database alias for tenant ("default" or tenant key)
    is_dry_run: bool               # Dry run flag (no DB writes)
    metadata: dict                 # Inter-stage data (query_ast, query_plan, compiled_queryset, observer)
    stash: dict                    # Custom stage-to-stage data storage
    model_cls: Any                 # Resolved Django model class
    model_metadata: Any            # Model registration metadata
    cache_plan: bool               # Whether to cache parsed query plan
```

**Computed Properties:**
- `user_id` — extracts `user.id` or `user.pk`
- `role` — extracts `user.role`
- `is_superadmin` — checks `user.is_superadmin` or `user.role.is_superadmin` or `user.is_superuser`
- `is_read` — `True` for read-type actions
- `is_mutation` — `True` for mutation actions
- `is_bulk` — `True` for bulk-prefixed actions

---

## 6. Multi-Tenancy Architecture

**Thread-safe context propagation** using Python's `ContextVar` (from `context.py`):

```python
current_tenant_db_var: ContextVar[str]  # Active tenant database alias
current_tenant_id_var: ContextVar[str]  # Active tenant ID
```

Each request's tenant is resolved from the authenticated user's token at Stage 04 (Authorization) and stored in `context.tenant_db`. All subsequent database operations route to the correct tenant database via `TenantDatabaseRouter`.

---

## 7. Pipeline Observer (Telemetry)

`PipelineObserver` (`populate_core/pipeline/observer.py`) provides per-stage performance timing:

- Activated by `settings.ENABLE_PIPELINE_OBSERVER = True` or `X-Observe: 1` request header or `?profile=1` query parameter
- Measures time spent in each stage using `measure_stage(stage_name)` context manager
- Logs bottleneck warnings if total execution exceeds 1.0 seconds
- Attaches timing data to the response (when detailed mode enabled)

---

## 8. URL Routing

**Entry point** (`populate_core/urls.py`):

```
/api/populate/health/                               → health check
/api/populate/<action>/<model_name>/                → standard actions
/api/populate/<action>/<model_name>/<object_id>/    → record-specific actions
```

The `PopulateAPIView` accepts all HTTP methods (GET, POST, PUT, PATCH, DELETE) for all action types — the HTTP method is not used to determine CRUD semantics. The `action` URL segment determines the operation.

---

## 9. Public Model Access

Certain model names bypass authentication (`AllowAny` permission) for read actions. This is implemented in `PopulateAPIView.get_permissions()` (`views.py`):

**Public models (verified from E-commerce instance):**
`policy_master`, `legal_policy`, `company`, `general_setting`, `hero_banner`, `category_master`, `product_type`, `product_variant`, `variant`, `product_image`, `product_review`, `promotion_coupon`, `faq_category`, `faq_item`, various master data, country/state/city masters, inventory stock (for storefront), print templates

**Special case:** `sale` model with `access_token`, `sale_number`, or `tracking_number` params — allows unauthenticated softcopy bill/order tracking.

**Public read responses get cache headers:**
```
Cache-Control: public, max-age=30, s-maxage=120, stale-while-revalidate=600
```

---

## 10. Consumer Integration

### Installation

```
# requirements.txt
git+ssh://git@github.com/Axinix-Technology/Axinix-Core.git#egg=django-populate-core

# Or for local development (via .env):
POPULATE_CORE_PATH="d:/Axinix Projects/Axinix-Core"
```

### settings.py Configuration

```python
INSTALLED_APPS = [
    'populate_core',   # Must be in INSTALLED_APPS
    'my_app',
]

POPULATE_CONFIG = {
    'ROLE_MODEL': 'users.Role',              # Your role model
    'SUPERADMIN_ROLE_FLAG': 'is_superadmin', # Superadmin boolean field name
    'MAX_PREFETCH_DEPTH': 3,                  # Max relation join depth
    'CHILD_PAGINATION_CEILING': 50,           # Max prefetched children
    'ENABLE_WORKFLOWS': True,                 # Pluggable workflow module
    'ENABLE_NOTIFICATIONS': True,             # Notification module
    'ENABLE_QUEUE': True,                     # Background queue module
}
```

### Model Registration

```python
from populate_core.registry import register_model

@register_model("ticket", table_type="transaction", status_field="status")
class Ticket(models.Model):
    title = models.CharField(max_length=255)
    status = models.SmallIntegerField(default=1)
```

### Domain Service Registration

```python
from populate_core.registry import BaseService, register_service

@register_service("ticket")
class TicketService(BaseService):
    def check_permission(self, context) -> bool | None:
        return None  # Defer to pipeline ABAC

    def before_create(self, context, data):
        return data  # Validate/transform data

    def after_create(self, context, instance):
        return instance  # Post-commit side effects
```

---

## 11. Security Architecture

| Layer | Mechanism | Location |
|---|---|---|
| **HTTP Authentication** | `IsAuthenticated` permission class | `views.py` |
| **Rate Limiting** | Redis sliding-window counter | `security/` module |
| **RBAC/ABAC** | `PolicyEngine` with `AccessPolicy` DB records | `policy/engine.py` |
| **Field-level security** | Allowlist/forbidden projections in sanitization | `security/sanitizer.py` |
| **Image decompression bombs** | `Pillow.MAX_IMAGE_PIXELS = 25_000_000` | Stage 03/06 |
| **Password length guard** | 128-character ceiling before bcrypt | Stage 01 |
| **MySQL query timeout** | `SET SESSION max_execution_time = 5000` | Query compiler |
| **Multi-tenant isolation** | `TenantDatabaseRouter` + `ContextVar` | `context.py` |
| **Audit logging** | Non-blocking `AuditLogger.log()` | Stage 10 |
| **Upload gate** | Authorization must pass before S3 upload | Stage 04 |

---

## 12. Known Constraints and Limitations

| Constraint | Value | Purpose |
|---|---|---|
| Max prefetch depth | 3 levels | Prevent memory exhaustion from deeply nested relations |
| Max child pagination | 50 items | Prevent OOM from large prefetch sets |
| MySQL query timeout | 5,000ms | Prevent runaway table scans from holding connections |
| Password input ceiling | 128 characters | Prevent Bcrypt CPU exhaustion DoS |
| Image pixel ceiling | 25,000,000 pixels | Prevent memory bomb from decompression |
| Rate limit ceiling | Redis-configured | Prevent API abuse |

---

## 13. Related Documents

- [Pipeline Stage Reference](./pipeline-stages-reference.md) — *(to be created)*
- [Service Hook Reference](./service-hooks-reference.md) — *(to be created)*
- [AccessPolicy Configuration](./access-policy-configuration.md) — *(to be created)*
- [../../../03-products/workhub/OVERVIEW.md](../../../03-products/workhub/) — Workhub product docs
- [REPOSITORY-INVENTORY.md](../../../00-discovery/REPOSITORY-INVENTORY.md)

---

## 14. Source File References

| Component | File |
|---|---|
| Pipeline executor | [`populate_core/pipeline/executor.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/pipeline/executor.py) |
| Stage implementations | [`populate_core/pipeline/stages.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/pipeline/stages.py) |
| Request context | [`populate_core/pipeline/context.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/pipeline/context.py) |
| View entry point | [`populate_core/views.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/views.py) |
| Policy engine | [`populate_core/policy/engine.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/policy/engine.py) |
| Audit logger | [`populate_core/audit/logger.py`](file:///d:/Axinix%20Projects/Axinix-Core/populate_core/audit/logger.py) |
| Package metadata | [`pyproject.toml`](file:///d:/Axinix%20Projects/Axinix-Core/pyproject.toml) |

