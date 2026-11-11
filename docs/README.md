# Axinix Technologies — Technical Knowledge Base

> **Organization:** Axinix Technologies  
> **Motto:** *"Innovating Future, Today"*  
> **Document Reference:** AXN-DOCS-ROOT  
> **Last Updated:** 2026-11-11  
> **Classification:** Internal Technical Documentation

---

## Purpose

This documentation system is the central, evidence-based technical knowledge base for Axinix Technologies.

Its purpose is to make the Axinix software ecosystem fully understandable without requiring someone to personally contact the original developer for every technical explanation.

**Methodology:**
- All documentation is produced by inspecting actual source code, not from memory or marketing descriptions.
- Every architectural claim is supported by a source-code reference.
- Implementation status is clearly distinguished from plans or proposals.
- Sensitive credentials and secret values are never documented.

---

## Ecosystem At a Glance

| Dimension | Detail |
|---|---|
| **Repositories** | 5 git repositories (`Axinix-Core`, `tracker-v2`, `E-commerce`, `Billing-ERP`, `AR.I.S`) |
| **Products** | 4 customer-facing products + 1 shared framework + 1 AI substrate |
| **Shared Framework** | Axinix Core (`django-populate-core`) — powers all Django backends |
| **Primary Backend** | Python / Django (all products) |
| **Primary Frontend** | React 19 + Vite |
| **Mobile** | Flutter (Workhub) |
| **Database** | MySQL 8.0+ (production), SQLite (dev/test) |
| **Cache/Queue** | Redis |
| **Cloud Storage** | AWS S3 |
| **AI/ML** | PyTorch (AR.I.S) |

---

## Documentation Structure

```text
docs/
│
├── README.md                       ← You are here
├── DOCUMENTATION-INDEX.md          ← Complete document index
├── DOCUMENTATION-STATUS.md         ← Progress tracker and gaps
│
├── 00-discovery/                   ← Phase 1: What exists
│   ├── REPOSITORY-INVENTORY.md     ← All repositories
│   ├── PRODUCT-INVENTORY.md        ← All products
│   ├── TECHNOLOGY-INVENTORY.md     ← All technologies
│   └── DEPENDENCY-INVENTORY.md     ← All dependencies
│
├── 01-organization/                ← Company and engineering overview
│
├── 02-shared-technology/           ← Reusable framework documentation
│   └── axinix-core/                ← Populate Engine deep documentation
│
├── 03-products/                    ← Product-specific documentation
│   ├── workhub/                    ← Workhub platform (tracker-v2)
│   ├── industry-applications/      ← E-commerce, Billing-ERP
│   └── internal-tools/             ← AR.I.S, internal utilities
│
├── 04-infrastructure/              ← Cloud, servers, deployment
│
├── 05-architecture/                ← Cross-system architecture diagrams
│
├── 06-engineering-standards/       ← Coding and architecture standards
│
├── 07-testing-quality/             ← Testing practices and quality
│
├── 08-deployment-operations/       ← Deployment and operations
│
├── 09-architecture-decisions/      ← ADRs (Architecture Decision Records)
│
└── 10-release-history/             ← Release notes and version history
```

---

## Quick Navigation

### By Product

| Product | Documentation |
|---|---|
| **Axinix Core** | [docs/02-shared-technology/axinix-core/](./02-shared-technology/axinix-core/) |
| **Workhub** | [docs/03-products/workhub/](./03-products/workhub/) |
| **E-Commerce Platform** | [docs/03-products/industry-applications/](./03-products/industry-applications/) |
| **Chai Point POS** | [docs/03-products/industry-applications/](./03-products/industry-applications/) |
| **AR.I.S** | [docs/03-products/internal-tools/](./03-products/internal-tools/) |

### By Topic

| Topic | Documentation |
|---|---|
| **API Architecture** | [docs/02-shared-technology/axinix-core/](./02-shared-technology/axinix-core/) |
| **Multi-tenancy** | [docs/02-shared-technology/axinix-core/](./02-shared-technology/axinix-core/) |
| **Security** | [docs/06-engineering-standards/](./06-engineering-standards/) |
| **Database Design** | [docs/03-products/workhub/](./03-products/workhub/) |
| **Infrastructure** | [docs/04-infrastructure/](./04-infrastructure/) |
| **Architecture Decisions** | [docs/09-architecture-decisions/](./09-architecture-decisions/) |

---

## Documentation Status

See [DOCUMENTATION-STATUS.md](./DOCUMENTATION-STATUS.md) for the current progress, gaps, and outstanding questions.

---

*Axinix Technologies — Confidential & Proprietary © 2026. All rights reserved.*
