# Technology Inventory

> **Document Reference:** AXN-DOCS-00-003  
> **Last Updated:** 2026-11-11  
> **Source:** Verified against `requirements.txt`, `pyproject.toml`, `pubspec.yaml`, `package.json` files across all repositories  
> **Authored By:** Axinix Technologies Documentation System

---

## Overview

This document inventories every verified technology, language, framework, and infrastructure component used across the Axinix Technologies software ecosystem.

---

## Programming Languages

| Language | Version | Usage | Repositories |
|---|---|---|---|
| **Python** | 3.11+ (Core), 3.12+ (E-commerce, Workhub) | Backend APIs, ML/AI, scripting | `Axinix-Core`, `tracker-v2`, `E-commerce`, `Billing-ERP`, `AR.I.S` |
| **JavaScript / JSX** | ES2022+ | Frontend web applications | `tracker-v2` (Frontend), `E-commerce` (frontend) |
| **Dart** | SDK `^3.10.4` | Mobile application | `tracker-v2` (tracker_mobile) |
| **Markdown** | Standard + GFM | Documentation | All repositories |

---

## Backend Frameworks and Libraries

### Django Ecosystem

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **Django** | `>=5.0` (Core), `6.1+` (E-commerce) | Web framework | `Axinix-Core`, `tracker-v2`, `E-commerce`, `Billing-ERP` |
| **Django REST Framework (DRF)** | `>=3.14.0` | REST API | `Axinix-Core`, `tracker-v2`, `E-commerce`, `Billing-ERP` |
| **django-populate-core** | `1.0.0` | Axinix Core pipeline kernel | `tracker-v2`, `E-commerce`, `Billing-ERP` |
| **django-redis** | `>=5.4.0` | Redis cache backend | `Axinix-Core` (optional dep) |
| **django-cors-headers** | `>=4.9.0` | CORS handling | `tracker-v2` |

### Authentication & Security

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **PyJWT** | `>=2.10.0` | JWT token generation/validation | `tracker-v2` |
| **cryptography** | `>=43.0.0` | Cryptographic operations | `tracker-v2` |
| **phonenumbers** | `>=8.13.0` | Phone number validation | `Axinix-Core` |

### Database Drivers

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **mysqlclient** | `>=2.2.0` | MySQL database driver (C extension) | `Axinix-Core` (optional), `tracker-v2`, `E-commerce` |
| **PyMySQL** | `>=1.1.0` | Pure-Python MySQL driver | `tracker-v2`, `AR.I.S` |

### Image Processing

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **Pillow** | `>=10.0.0` | Image processing, decompression bomb protection | `Axinix-Core` (optional), `E-commerce` |

### Data Export

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **openpyxl** | `>=3.1.0` | Excel export | `Axinix-Core` (full optional) |

### Configuration

| Library | Version | Purpose | Repositories |
|---|---|---|---|
| **python-dotenv** | `>=1.0.0` – `>=1.2.4` | `.env` file loading | `tracker-v2`, `AR.I.S` |

---

## AI / Machine Learning Libraries (AR.I.S)

| Library | Version | Purpose |
|---|---|---|
| **PyTorch** | `>=2.0.0` | Neural network training and inference |
| **HuggingFace Tokenizers** | `>=0.20.0` | Tokenizer training and inference |
| **HuggingFace Datasets** | `>=2.14.0` | Dataset loading and processing |
| **HuggingFace Hub** | `>=0.20.0` | Model and dataset downloads |
| **NumPy** | `>=1.24.0` | Numerical computing |
| **PyArrow** | `>=14.0.0` | Columnar data processing |
| **tqdm** | `>=4.66.0` | Progress bars |
| **requests** | `>=2.31.0` | HTTP client |

---

## Frontend Web Technologies

| Technology | Version | Purpose | Repositories |
|---|---|---|---|
| **React** | `19.0.0` | UI component library | `tracker-v2`, `E-commerce` |
| **Vite** | `6.x` | Build tool and dev server | `tracker-v2`, `E-commerce` |
| **Tailwind CSS** | `v4.0` | Utility-first CSS framework | `E-commerce` |
| **Axios** | (detected from E-commerce) | HTTP client for API calls | `E-commerce` |
| **React (legacy/node era)** | `18.x` | Retained from Node.js era | `Billing-ERP` |

> [!NOTE]
> Exact versions for `tracker-v2` frontend dependencies require reading `Frontend/package.json`.

---

## Mobile Technologies (Workhub)

| Technology | Version | Purpose |
|---|---|---|
| **Flutter** | SDK `^3.10.4` | Cross-platform mobile framework |
| **Dart** | (bundled with Flutter) | Programming language |
| **Dio** | `^5.9.2` | HTTP client for Flutter |
| **Provider** | `^6.0.5` | State management |
| **Camera** | `^0.12.0` | Device camera access |
| **Image Picker** | `^1.1.2` | Gallery/camera image selection |
| **JWT Decoder** | `^2.0.1` | JWT token decoding |
| **Shared Preferences** | `^2.2.1` | Local key-value storage |
| **Connectivity Plus** | `^7.0.0` | Network connectivity detection |
| **Shimmer** | `any` | Loading skeleton UI |
| **Lucide Icons Flutter** | `^3.1.14+1` | Icon library |
| **flutter_svg** | `^2.2.4` | SVG rendering |
| **cached_network_image** | `^3.4.1` | Image caching |
| **dio_cookie_manager** | `^3.4.0` | Cookie management for Dio |
| **intl** | `^0.20.2` | Internationalization |

---

## Infrastructure and Cloud Technologies

| Technology | Purpose | Repositories |
|---|---|---|
| **MySQL 8.0+** | Primary relational database (production) | `tracker-v2`, `E-commerce`, `Billing-ERP` |
| **SQLite** | Development/testing database | `Billing-ERP` (dev), `Axinix-Core` (test suite uses in-memory SQLite) |
| **Redis** | Rate limiting, caching, task queue | `Axinix-Core`, `tracker-v2` |
| **AWS S3** | Media and file storage | `E-commerce` |
| **Aiven MySQL** | Managed cloud MySQL (SSL) | `E-commerce` (production) |

---

## Development and Build Tools

| Tool | Purpose | Repositories |
|---|---|---|
| **Git** | Version control | All repositories |
| **npm** | Node.js package manager | `tracker-v2`, `E-commerce`, `Billing-ERP` (legacy) |
| **pip** | Python package manager | `Axinix-Core`, `tracker-v2`, `E-commerce`, `Billing-ERP`, `AR.I.S` |
| **pytest** / **pytest-django** | Python testing | `Axinix-Core` |
| **setuptools** / **wheel** | Python package building | `Axinix-Core` |
| **PowerShell** | Deployment scripting | `tracker-v2` (`deploy.ps1`) |
| **GitHub Actions** | CI/CD (branch activity suggests) | `tracker-v2`, `E-commerce` |

---

## Security Architecture Technologies

| Component | Technology | Usage |
|---|---|---|
| **RBAC/ABAC Policy Engine** | Custom (Axinix Core) | All Workhub, E-commerce, Billing-ERP backends |
| **Multi-tenant DB Router** | `TenantDatabaseRouter` (Axinix Core) | Workhub |
| **Token Authentication** | DRF Token Auth | E-commerce, Workhub |
| **JWT Authentication** | PyJWT | Workhub backend |
| **SHA-256 Pre-hashing** | Browser `crypto.subtle` | E-commerce login |
| **Dual Password Backend** | Custom Django auth backend | E-commerce |
| **Redis Rate Limiting** | Sliding window counter | Axinix Core |
| **Image Decompression Protection** | `Pillow.MAX_IMAGE_PIXELS` | Axinix Core (Stage 06) |
| **MySQL Query Timeout** | `SET SESSION max_execution_time=5000` | Axinix Core |
| **Password Length Guard** | 128-char ceiling | Axinix Core (Stage 01) |

---

## Related Documents

- [REPOSITORY-INVENTORY.md](./REPOSITORY-INVENTORY.md)
- [PRODUCT-INVENTORY.md](./PRODUCT-INVENTORY.md)
- [DEPENDENCY-INVENTORY.md](./DEPENDENCY-INVENTORY.md)
