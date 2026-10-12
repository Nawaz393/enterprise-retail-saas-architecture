# Architecture: Bounded Contexts & Inversion of Control (IoC)

To manage 120,000+ lines of TypeScript and prevent cyclical dependencies, the SAK Retail Solutions backend strictly enforces Domain-Driven Design (DDD). The system is partitioned into 19 isolated domain packages.

## Container & Component Topology

```mermaid
flowchart TD
    subgraph HTTP ["HTTP Transport Layer"]
        Router["AdonisJS v6 Router"] --> Validator["VineJS Schema Validator"]
    end

    subgraph OrderDomain ["#order/* Bounded Context"]
        Validator --> Ctrl["CheckoutController"]
        Ctrl --> Svc["CheckoutService<br/>(@inject IoC Container)"]
    end

    subgraph ExternalDomains ["Injected Domain Services"]
        Svc -->|"this.cartService"| CartSvc["#cart/Services/CartService"]
        Svc -->|"this.pricingService"| TaxSvc["#tax/Services/PricingService"]
        Svc -->|"this.stockService"| ProductSvc["#product/Services/StockService"]
    end

    CartSvc --> DB[("PostgreSQL 16")]
    TaxSvc --> DB
    ProductSvc --> DB
