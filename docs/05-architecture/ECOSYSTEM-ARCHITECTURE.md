# Axinix Ecosystem — Architecture Diagrams

> **Document Reference:** AXN-DOCS-05-ARCH-001  
> **Last Updated:** 2026-11-11  
> **Authored By:** Axinix Technologies Documentation System  
> **Source:** Evidence-based from repository inspection

---

## 1. Axinix Product Ecosystem Map

```mermaid
graph TD
    subgraph Foundation ["🏗️ Shared Foundation Layer"]
        CORE["Axinix Core<br/>(django-populate-core v1.0.0)<br/>10-Stage Pipeline Kernel"]
    end

    subgraph SaaS ["📦 SaaS Product Layer"]
        WORKHUB["Workhub v4.1.1<br/>tracker-v2<br/>HRMS · Payroll · CRM · Tasks · Tickets · Billing"]
        ECOMM["E-Commerce v1.1.1<br/>E-commerce<br/>Catalog · Inventory · Orders · Channels"]
        BILLING["Chai Point POS<br/>Billing-ERP<br/>POS · Invoicing · Analytics"]
    end

    subgraph AI ["🧠 AI Layer"]
        ARIS["AR.I.S<br/>Artificial Intelligent System<br/>Neural Language Model Substrate"]
    end

    subgraph Corp ["🏢 Corporate"]
        GITHUB[".github<br/>Org Profile + Docs<br/>Corporate Portfolio"]
    end

    CORE -->|"pip install git+ssh"| WORKHUB
    CORE -->|"pip install git+ssh"| ECOMM
    CORE -->|"pip install git+"| BILLING

    ARIS -.->|"Future: Jarvis AI Assistant"| WORKHUB
    ARIS -.->|"Future: AI-powered features"| ECOMM
```

---

## 2. Axinix Core — 10-Stage Request Pipeline

```mermaid
sequenceDiagram
    participant C as Client (Browser/App)
    participant V as PopulateAPIView
    participant P as PopulatePipeline

    C->>V: HTTP Request (GET/POST/PUT/PATCH/DELETE)<br/>/api/populate/{action}/{model}/

    V->>V: get_permissions()<br/>IsAuthenticated or AllowAny (public models)

    V->>P: dispatch_pipeline(request, action, model_name, object_id)

    Note over P: Stage 01 — INGRESS
    P->>P: IngressStage.execute()<br/>Parse request → RequestContext<br/>(action, body, filters, populate, pagination,<br/>user, request_id, ip_address)

    Note over P: Stage 02 — VALIDATION
    P->>P: ValidationStage.execute()<br/>RequestValidator.validate(context)<br/>Schema and shape check

    Note over P: Stage 03 — SANITIZATION
    P->>P: SanitizationStage.execute()<br/>SecuritySanitizer.sanitize(context)<br/>Identifier and parameter safety

    Note over P: Stage 04 — AUTHORIZATION
    P->>P: AuthorizationStage.execute()<br/>PolicyEngine.authorize(context)<br/>RBAC/ABAC — Fail Closed on reject

    Note over P: Stage 05 — PARSING
    P->>P: ParsingStage.execute()<br/>DeveloperDSLParser.parse(context)<br/>JSON filter DSL → AST

    Note over P: Stage 06 — PLANNING
    P->>P: PlanningStage.execute()<br/>QueryPlanner.plan(ast, model_cls)<br/>Join and prefetch planning

    Note over P: Stage 07 — COMPILATION
    P->>P: CompilationStage.execute()<br/>QueryCompiler.compile(plan, context)<br/>AST → Django QuerySet

    Note over P: Stage 08 — EXECUTION
    P->>P: ExecutionStage.execute()<br/>ReadExecutor / CreateExecutor / UpdateExecutor<br/>/ DeleteExecutor / ReportExecutor<br/>+ Domain Service Hooks (before_*, after_*)

    Note over P: Stage 09 — SERIALIZATION
    P->>P: SerializationStage.execute()<br/>OutputSerializer.serialize_result()<br/>Output security + field projections

    Note over P: Stage 10 — FINALIZATION
    P->>P: FinalizationStage.execute_success()<br/>AuditLogger.log() + success_response()<br/>X-Request-ID header

    P->>V: HTTP Response {success, data, count, metadata}
    V->>C: JSON Response
```

---

## 3. Repository Relationships

```mermaid
graph LR
    subgraph Repos ["Repositories"]
        CORE["Axinix-Core<br/>github.com/Axinix-Technology/Axinix-Core"]
        TRACKER["tracker-v2<br/>(Workhub)"]
        ECOMM["E-commerce"]
        BILLING["Billing-ERP"]
        ARIS["AR.I.S"]
        GITHUB[".github<br/>(Org + Docs)"]
    end

    CORE -->|"requirements.txt<br/>git+ssh pip install"| TRACKER
    CORE -->|"requirements.txt<br/>git+ssh pip install"| ECOMM
    CORE -->|"requirements.txt<br/>git+ pip install"| BILLING
```

---

## 4. Workhub Multi-Tenant Architecture

```mermaid
flowchart TD
    subgraph Internet ["Internet"]
        CLIENT["User Browser / Mobile App"]
    end

    subgraph Frontend ["Frontend Layer"]
        REACT["React 19 + Vite<br/>(Frontend/src/)"]
        FLUTTER["Flutter Mobile App<br/>(tracker_mobile/)"]
    end

    subgraph Backend ["Backend — Django + Axinix Core"]
        AUTH["Auth Endpoints<br/>users/urls.py<br/>/auth/login → /auth/me"]
        POPULATE["Populate Engine<br/>/api/populate/{action}/{model}/"]
        SERVICE["Domain Services<br/>(services.py per module)"]
    end

    subgraph DataLayer ["Data Layer"]
        GLOBAL["Global DB (default)<br/>control_plane tables:<br/>Tenant, UserLogin, Module"]
        T1["Tenant DB: tracker_axinix<br/>All module tables"]
        T2["Tenant DB: tracker_clientxyz<br/>All module tables"]
        REDIS["Redis<br/>Rate limit · Cache · Queue"]
    end

    CLIENT --> REACT
    CLIENT --> FLUTTER
    REACT -->|"POST /auth/login"| AUTH
    REACT -->|"POST /api/populate/*"| POPULATE
    FLUTTER -->|"dio HTTP"| AUTH
    FLUTTER -->|"dio HTTP"| POPULATE
    AUTH -->|"Lookup UserLogin"| GLOBAL
    POPULATE --> SERVICE
    SERVICE -->|"TenantDatabaseRouter<br/>routes to correct DB"| T1
    SERVICE -->|"TenantDatabaseRouter<br/>routes to correct DB"| T2
    POPULATE --> REDIS
```

---

## 5. Axinix Core Security Architecture

```mermaid
flowchart LR
    REQ["Incoming Request"] --> S01["Stage 01<br/>INGRESS<br/>Build RequestContext<br/>+ Extract X-Request-ID"]
    S01 --> S02["Stage 02<br/>VALIDATION<br/>Schema shape check"]
    S02 --> S03["Stage 03<br/>SANITIZATION<br/>Strip forbidden fields<br/>Injection prevention"]
    S03 --> S04["Stage 04<br/>AUTHORIZATION<br/>PolicyEngine RBAC/ABAC<br/>2-Tier Permission Ladder"]
    S04 -->|"FAIL → 401/403"| ERR["FinalizationStage<br/>execute_error()\nAuditLogger.log()"]
    S04 -->|"PASS"| S05["Stage 05-07<br/>PARSING → PLANNING<br/>→ COMPILATION<br/>AST → QuerySet"]
    S05 --> S08["Stage 08<br/>EXECUTION<br/>Domain Service Hooks<br/>before_* / after_*"]
    S08 --> S09["Stage 09<br/>SERIALIZATION<br/>Field allowlists<br/>Output security"]
    S09 --> S10["Stage 10<br/>FINALIZATION<br/>AuditLogger.log()<br/>X-Request-ID header"]
```

---

## 6. E-Commerce Platform Architecture

```mermaid
flowchart TD
    subgraph FE ["Frontend (React 19 + Vite 6)"]
        ADMIN["Admin Console"]
        PROXY["Vite Proxy (/api → :8000)"]
    end

    subgraph BE ["Backend (Django 6.1 + DRF)"]
        POPULATE["/api/populate/<br/>Axinix Core Engine"]
        CUSTOM["/api/v1/users/login<br/>/api/company/public/"]
        DOMAINS["Domain Apps:<br/>catalogue · inventory · orders<br/>channels · storefront · inward<br/>company · users · integrations"]
    end

    subgraph DB ["Database"]
        LOCAL["Local MySQL<br/>(DEBUG=True)"]
        CLOUD["Aiven Cloud MySQL SSL<br/>(DEBUG=False)"]
    end

    subgraph STORAGE ["Storage"]
        S3["AWS S3<br/>(Media & Images)"]
    end

    ADMIN --> PROXY
    PROXY --> POPULATE
    PROXY --> CUSTOM
    POPULATE --> DOMAINS
    CUSTOM --> DOMAINS
    DOMAINS -->|"DEBUG=True"| LOCAL
    DOMAINS -->|"DEBUG=False"| CLOUD
    DOMAINS --> S3
```

---

## 7. AR.I.S Neural Architecture

```mermaid
flowchart TD
    subgraph Data ["Training Data (data/)"]
        GSM8K["GSM8K<br/>Math reasoning"]
        MBPP["MBPP<br/>Code generation"]
        HOTPOT["HotpotQA<br/>Multi-hop QA"]
        QASRL["qasrl_gs<br/>Linguistic / SRL"]
        ULTRA["UltraChat 200k<br/>Conversations"]
        VISION["Vision<br/>Computer vision"]
    end

    subgraph Core ["Core System (aris/)"]
        CONFIG["configs/<br/>Configuration mgmt"]
        TOKENIZER["tokenizer/<br/>Custom tokenizer"]
        TRAINING["training/<br/>Training pipeline"]
        INFERENCE["inference/<br/>Inference engine"]
        LINGUISTIC["linguistic/<br/>CognitiveLinguisticFrame"]
        MEMORY["memory/<br/>Memory system"]
        EVAL["evaluation/<br/>Evaluation suite"]
    end

    subgraph Output ["Trained Models (models/)"]
        TRAINED["Neural model artifacts"]
    end

    Data -->|"Ingested via data/"| TRAINING
    TOKENIZER --> TRAINING
    TRAINING --> TRAINED
    TRAINED --> INFERENCE
    LINGUISTIC --> INFERENCE
    MEMORY --> INFERENCE
    INFERENCE --> EVAL

    TRAINED -.->|"Future: Powers"| WORKHUB["Workhub<br/>(Jarvis AI)"]
    TRAINED -.->|"Future: Powers"| ECOMM["E-Commerce<br/>(AI features)"]
```

---

## 8. Deployment Architecture (Workhub)

```mermaid
flowchart LR
    subgraph Dev ["Development"]
        LOCAL_FE["Vite Dev Server :5173"]
        LOCAL_BE["Django :8000"]
        LOCAL_DB["Local MySQL"]
        LOCAL_REDIS["Local Redis"]
    end

    subgraph Deploy ["Deployment (inferred from deploy.ps1)"]
        PS["deploy.ps1<br/>PowerShell Script"]
        SERVER["Production Server<br/>(AWS EC2 — from Arun/notes)"]
    end

    LOCAL_FE -->|"Vite proxy /api"| LOCAL_BE
    LOCAL_BE --> LOCAL_DB
    LOCAL_BE --> LOCAL_REDIS
    PS -->|"Deploy"| SERVER
```

> [!NOTE]
> Production deployment details (server IPs, hostnames, SSL certificates, AWS configuration) are referenced in `Arun/Amazon Cloud.txt` but are not documented here for security reasons. Contact the DevOps team (Rithika, Co-Founder DevOps Head) for production infrastructure access.

---

## Related Documents

- [../00-discovery/REPOSITORY-INVENTORY.md](../00-discovery/REPOSITORY-INVENTORY.md)
- [../02-shared-technology/axinix-core/ARCHITECTURE.md](../02-shared-technology/axinix-core/ARCHITECTURE.md)
- [../03-products/workhub/OVERVIEW.md](../03-products/workhub/OVERVIEW.md)

