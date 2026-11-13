# Axinix E-Commerce Platform — Overview

> **Document Reference:** AXN-DOCS-03-IND-001  
> **Last Updated:** 2026-11-11  
> **Repository:** `d:\Axinix Projects\E-commerce`  
> **Version:** `1.1.1`  
> **Implementation Status:** ACTIVELY DEVELOPED  
> **Authored By:** Axinix Technologies Documentation System

---

## 1. Overview

The **Axinix Centralized E-Commerce Platform** (`central-ecommerce-platform`) is an enterprise omnichannel inventory and billing solution for manufacturers, brands, and multi-channel retailers. It provides centralized catalog management, inventory tracking, POS billing, order fulfillment, channel synchronization, and storefront services.

**Engineering motto:** *"Engineering Tomorrow!"*

---

## 2. Platform Specifications

| Dimension | Detail |
|---|---|
| **Version** | `1.1.1` |
| **Backend** | Python / Django 6.1 + DRF + Axinix Core |
| **Frontend** | React 19 + Vite 6 + Tailwind CSS v4 |
| **Database** | MySQL 8.0+ (Aiven cloud SSL in production, local in dev) |
| **Media Storage** | AWS S3 |
| **Auth** | DualPasswordBackend + DRF Token |
| **API Pattern** | Axinix Core populate engine (`/api/populate/`) + custom auth/company endpoints |

---

## 3. Business Capabilities

| Capability | Implementation | Status |
|---|---|---|
| Cloud POS & Retail Billing | `backend/billing/` (via Core) | ✅ IMPLEMENTED |
| Centralized Catalog & SKU | `backend/catalogue/` | ✅ IMPLEMENTED |
| Inventory Tracking | `backend/inventory/` | ✅ IMPLEMENTED |
| Inward/Stock Receiving | `backend/inward/` | ✅ IMPLEMENTED |
| Order Management | `backend/orders/` | ✅ IMPLEMENTED |
| Order Cancellation Flow | `backend/orders/` (recent) | ✅ IMPLEMENTED |
| Omnichannel Channel Sync | `backend/channels/` | ✅ IMPLEMENTED |
| Storefront Services | `backend/storefront/` | ✅ IMPLEMENTED |
| Company Master & Branding | `backend/company/` | ✅ IMPLEMENTED |
| Media/Image Management | `backend/media/` + AWS S3 | ✅ IMPLEMENTED |
| External Integrations | `backend/integrations/` | ✅ IMPLEMENTED |
| Barcode Label Generation | `spooler/` (auto-print) | ✅ IMPLEMENTED |
| COD Delivery Agent Hub | `backend/orders/` (recent) | ✅ IMPLEMENTED |
| PDF/Print Invoices | Via `print_template` model | ✅ IMPLEMENTED |

---

## 4. Authentication System

**DualPasswordBackend** (`backend/users/backends.py`):
- Enables authentication from both React UI (SHA-256 pre-hashed) and Django Admin/API tools (raw password)
- Client-side: `crypto.subtle.digest("SHA-256")` before transmitting credentials
- Server-side: accepts both SHA-256 pre-hash and plain password
- DRF Token Auth for subsequent requests
- `UserSession` model tracks login devices, IPs, and FCM push tokens

---

## 5. Database Configuration

**Dual-mode database switching** (from README + settings.py inspection):

```
DEBUG=True  → Local MySQL (development)
DEBUG=False → Aiven Cloud MySQL SSL (production)
```

| Setting | Variable | Usage |
|---|---|---|
| Local MySQL | `MYSQL_HOST/PORT/USER/PASSWORD/DATABASE` | Development |
| Cloud MySQL | `MYSQL_DATABASE_LIVE` (full connection URI) | Production |

---

## 6. Known Domain Models (from backend directory)

| App | Purpose | Axinix Core Pattern |
|---|---|---|
| `catalogue` | Products, variants, categories | `@register_model` |
| `inventory` | Stock tracking, SKU ledger | `@register_model` |
| `inward` | Inward batches, supplier receipts | `@register_model` |
| `orders` | Customer orders, fulfillment, cancellation | `@register_model` |
| `channels` | Marketplace channels (Shopify, etc.) | `@register_model` |
| `storefront` | Storefront API (product listings, cart) | `@register_model` |
| `company` | Company master, branding, settings | `@register_model` |
| `users` | RBAC roles, user accounts | `@register_model` |
| `integrations` | External API integrations | `@register_model` |
| `media` | Media file management | `@register_model` |

---

## 7. Default System Roles

| Role | `is_superadmin` | Access |
|---|---|---|
| **Super Admin** | True | Full platform administration |
| **Store Manager** | False | Catalog, inventory, order fulfillment |
| **Staff** | False | Warehouse and retail operations |
| **Customer** | False | Storefront purchasing |

---

## 8. Essential API Endpoints

| Method | Endpoint | Purpose | Auth |
|---|---|---|---|
| `POST` | `/api/v1/users/login/` | Authenticate and get token | ❌ |
| `GET` | `/api/v1/users/me/` | Current user profile | ✅ |
| `GET` | `/api/v1/company/public/` | Public company branding | ❌ |
| `POST` | `/api/populate/{action}/{model}/` | All CRUD via Core engine | ✅ (or ❌ for public models) |
| `GET` | `/api/test/` | Health check + DB status | ❌ |
| `GET` | `/admin/` | Django admin portal | ✅ |

---

## 9. Public Models (unauthenticated read access)

From `views.py` inspection (Axinix Core's `get_permissions()`):
- Company information, general settings, hero banners
- Category master, product types, variants, images
- Product reviews, promotional coupons, FAQs
- Master data (materials, designs, brands, sizes, age groups)
- Geographic data (countries, states, cities)
- Inventory stock (for storefront display)
- Order tracking milestones
- Print templates

**Special:** `sale` model — unauthenticated access with `access_token` or `sale_number` or `tracking_number` (for softcopy bill and order tracking)

---

## 10. Source File References

| Component | File |
|---|---|
| README | [`README.md`](file:///d:/Axinix%20Projects/E-commerce/README.md) |
| Core API view | [`backend/core/`](file:///d:/Axinix%20Projects/E-commerce/backend/core/) (requires deeper scan) |
| Version metadata | [`version.json`](file:///d:/Axinix%20Projects/E-commerce/version.json) |
| Backend structure | [`backend/`](file:///d:/Axinix%20Projects/E-commerce/backend/) |

---

## Related Documents

- [Chai Point POS Overview](./chai-point-pos/OVERVIEW.md) — *(to be created)*
- [../../../02-shared-technology/axinix-core/ARCHITECTURE.md](../../02-shared-technology/axinix-core/ARCHITECTURE.md)
- [REPOSITORY-INVENTORY.md](../../../00-discovery/REPOSITORY-INVENTORY.md)

