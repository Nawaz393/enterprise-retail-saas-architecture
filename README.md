# 🛒 Enterprise Omnichannel Retail & POS Infrastructure

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![AdonisJS](https://img.shields.io/badge/AdonisJS-v6-5A45FF.svg)](https://adonisjs.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB.svg)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791.svg)](https://www.postgresql.org/)
[![AWS ECS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900.svg)](https://aws.amazon.com/ecs/)
[![PCI--DSS](https://img.shields.io/badge/PCI--DSS-SAQ--A%20Compliant-green.svg)](https://www.pcisecuritystandards.org/)

## Executive Architecture Summary
This repository outlines the system design for a flagship multi-tenant Point of Sale (POS) and retail infrastructure platform. Spanning over 120,000 lines of production TypeScript, the system orchestrates POS hardware procurement, multi-tiered Independent Sales Organization (ISO) distribution networks, real-time inventory locking, and centralized financial ledgering.

The backend is engineered on **Node.js 22 LTS** using a **Domain-Driven Design (DDD)** architecture, ensuring bounded contexts across 19 distinct operational domains (Auth, RBAC, Merchant, ISO, Product, Order, Payment, Tax, Commission).

## 1. Domain-Driven Design & Bounded Contexts
To eliminate cyclical dependencies and tightly coupled business logic, the application layer is strictly partitioned:
* **Inversion of Control (IoC):** All domain controllers, policies, and services utilize strict constructor dependency injection (`@inject()`).
* **ESM Subpath Encapsulation:** Cross-domain imports are mapped via `package.json` subpaths (`#auth/*`, `#order/*`, `#payment/*`), enforcing structural boundaries and blocking deep imports into private internals.
* **Type-Safe Validation:** Runtime inputs are validated against compiled VineJS schemas to guarantee structural integrity before hitting domain services.

## 2. Multi-Tenant Security & Zero-Trust Data Isolation
The platform serves a three-tier hierarchy: Global Admins, Regional ISOs, and Individual Merchants.
* **Fail-Closed SQL Query Scoping:** Policies enforce data isolation at the AST/database query layer. The `applyScope()` method dynamically binds `WHERE` clauses based on tenant context (`merchant.view_iso`, `merchant.view_self`)[cite: 2]. If no valid scope matches, it appends `1 = 0` to physically prevent data leakage at the database level[cite: 2].
* **Sub-Millisecond RBAC Caching:** User permissions and role expirations are cached in high-performance in-memory Maps (`PermissionCacheService`), eliminating database round-trips for authorization checks on every HTTP request[cite: 2].

## 3. Deterministic Financial Engine
Floating-point approximations (IEEE-754) are unacceptable in financial ledgering.
* **Arbitrary-Precision Arithmetic:** All pricing, commission, and tax computations utilize `Decimal.js` (`decimal(15, 2)`)[cite: 2].
* **Proportional Bundle Apportionment:** When POS hardware bundles are sold at promotional discounts, the system calculates the proportional ratio of each sub-component to the total list price, ensuring accurate jurisdictional tax apportioning without fractional-cent rounding drift[cite: 2].

## 4. PostgreSQL 16 Database Invariants
Application-layer validation is bypassed by concurrent workers or direct database migrations. Structural consistency is pushed down to the storage engine across 58 migrations[cite: 2]:
* **Partial Unique Indexes:** Guarantees that each catalog product has exactly one active default variant at the database level (`WHERE is_default AND NOT is_deleted`)[cite: 2].
* **XOR Polymorphic Constraints:** `CHECK (num_nonnulls(product_id, product_item_id) = 1)` ensures assets link to either a top-level product or a variant item, preventing orphan records[cite: 2].
* **Deferrable Triggers:** `enforce_product_item_default()` evaluates variant defaults at the end of atomic transactions (`DEFERRABLE INITIALLY DEFERRED`) to safely handle transitional states during variant swaps[cite: 2].
* **Automated JSONB Auditing:** PL/pgSQL triggers automatically capture before-and-after state diffs for all mutations, writing to an immutable `audit_log` without introducing application latency[cite: 2].

## 5. Zero-PCI Surface Area & Event-Driven Workflows
* **PCI-DSS SAQ-A Compliance:** Credit card PANs and CVVs bypass our application servers entirely[cite: 2]. Authorize.Net `Accept.js` handles client-side tokenization, providing the backend with a single-use opaque payment nonce[cite: 2].
* **Asynchronous Microservices:** Upon a successful atomic checkout transaction (`db.transaction`), the API emits an `order:paid` event[cite: 2]. This queues jobs via **BullMQ and Redis 7** to an isolated worker microservice (`sakrs-email-service`)[cite: 2], which handles CPU-bound PDF invoice generation and Amazon SESv2 dispatch[cite: 2].
