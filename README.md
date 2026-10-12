# Enterprise Omnichannel Retail & POS Infrastructure

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-22_LTS-339933.svg)](https://nodejs.org/)
[![Framework](https://img.shields.io/badge/AdonisJS-v6-5A45FF.svg)](https://adonisjs.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB.svg)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791.svg)](https://www.postgresql.org/)
[![AWS ECS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900.svg)](https://aws.amazon.com/ecs/)
[![PCI-DSS](https://img.shields.io/badge/PCI--DSS-SAQ--A%20Compliant-green.svg)](https://www.pcisecuritystandards.org/)

## Executive Architecture Summary

This repository details the system design, core abstractions, and database invariants of an enterprise-grade, multi-tenant Point of Sale (POS) and retail infrastructure platform. Spanning over 120,000 lines of production TypeScript, the system orchestrates POS hardware procurement, multi-tiered Independent Sales Organization (ISO) agent networks, real-time inventory locking, and centralized financial ledgering.

The backend is engineered on Node.js 22 LTS utilizing a strict Domain-Driven Design (DDD) architecture. The platform completely eliminates IEEE-754 floating-point drift, enforces row-level security directly at the SQL Abstract Syntax Tree (AST) layer, and utilizes a zero-PCI surface area approach for cryptographic tokenization.

## Architecture Documentation Library

The system design is documented extensively across the following domains. Please refer to the specific whitepapers below for architectural decisions, mathematical formulas, and production code implementations:

1. [Domain-Driven Design (DDD) & Inversion of Control](./docs/01-domain-driven-design.md)
2. [Fail-Closed Multi-Tenant Row-Level Security](./docs/02-multi-tenant-security.md)
3. [Deterministic Financial Math & Bundle Apportionment](./docs/03-financial-math-engine.md)
4. [Distributed Checkout Orchestration & ACID Sagas](./docs/04-transactional-checkout.md)
5. [Storage Engine Invariants & CDC Auditing](./docs/05-storage-engine-invariants.md)

---

## Cloud Topology & Distributed Network Mesh

Enterprise retail systems require absolute network isolation between public-facing ingress and stateful ledger systems. The topology leverages AWS ECS Fargate with a private Service Connect mesh to ensure that database, caching, and background worker containers have zero direct public IP exposure.

```mermaid
flowchart TD
    subgraph PublicInternet ["Public Internet Ingress"]
        ClientSPA["Client Browser (React 19 SPA)"]
        VaultAPI["Payment Gateway Vault (Tokenization)"]
    end

    subgraph AWSCloud ["AWS VPC (Production Region)"]
        ALB["AWS Application Load Balancer (ALB)<br/>TLS 1.3 Termination"]

        subgraph PrivateSubnet ["Private Isolated VPC Subnet"]
            subgraph FargateCluster ["AWS ECS Fargate Cluster"]
                APISvc["Backend API Service<br/>(Node 22 LTS / Pure ESM / Port 3333)"]
                WorkerSvc["Email & PDF Worker Microservice<br/>(Background Job Worker / Port 3334)"]
            end

            subgraph StorageDataTier ["Data Tier & State Synchronization"]
                RedisEngine[("Redis 7 Alpine Cluster<br/>Job Queues & Permission Cache")]
                RDSPG[("Amazon RDS PostgreSQL 16<br/>Multi-AZ / 58 Migrations")]
                S3Storage[("Amazon S3 Bucket<br/>Encrypted Invoices & Assets")]
            end
            
            SecretsMgr["AWS Secrets Manager"]
        end
    end

    ClientSPA -->|"HTTPS / Encrypted Card Data"| VaultAPI
    VaultAPI -->|"Single-Use Nonce (dataValue)"| ClientSPA
    ClientSPA -->|"JSON Payloads + Bearer Token"| ALB
    ALB -->|"HTTP Forwarding"| APISvc

    APISvc -->|"Service Connect / TCP 6379"| RedisEngine
    WorkerSvc -->|"Fetch Queued Tasks"| RedisEngine
    APISvc -->|"ORM / PgPool (Port 5432)"| RDSPG
    APISvc -->|"Multipart Put / Fetch Stream"| S3Storage
    WorkerSvc -->|"Read Invoice Binary Stream"| S3Storage
    
    APISvc -.->|"IAM Role Secret Fetch"| SecretsMgr
    WorkerSvc -.->|"IAM Role Secret Fetch"| SecretsMgr
