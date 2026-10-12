# Enterprise Omnichannel Retail & POS Infrastructure

[![TypeScript](https://img.shields.io/badge/TypeScript-5.8-blue.svg)](https://www.typescriptlang.org/)
[![AdonisJS](https://img.shields.io/badge/AdonisJS-v6-5A45FF.svg)](https://adonisjs.com/)
[![React](https://img.shields.io/badge/React-19-61DAFB.svg)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791.svg)](https://www.postgresql.org/)
[![AWS ECS](https://img.shields.io/badge/AWS-ECS%20Fargate-FF9900.svg)](https://aws.amazon.com/ecs/)
[![PCI-DSS](https://img.shields.io/badge/PCI--DSS-SAQ--A%20Compliant-green.svg)](https://www.pcisecuritystandards.org/)

## Executive Architecture Summary

This document details the system design, core abstractions, and database invariants of an enterprise-grade, multi-tenant Point of Sale (POS) and retail infrastructure platform. Spanning over 120,000 lines of production TypeScript, the system orchestrates POS hardware procurement, multi-tiered Independent Sales Organization (ISO) agent networks, real-time inventory locking, and centralized financial ledgering. 

The backend is engineered on Node.js 22 LTS utilizing a strict Domain-Driven Design (DDD) architecture. The platform completely eliminates IEEE-754 floating-point drift, enforces row-level security directly at the SQL Abstract Syntax Tree (AST) layer, and utilizes a zero-PCI surface area approach for cryptographic tokenization.

---

## 1. Cloud Topology & Distributed Network Mesh

### Engineering Rationale
Enterprise retail systems require absolute network isolation between public-facing ingress and stateful ledger systems. The topology leverages AWS ECS Fargate with a private Service Connect mesh to ensure that database, caching, and background worker containers have zero direct public IP exposure.

### Container Architecture Diagram (C4 Level 2)
```mermaid
flowchart TD
    subgraph PublicInternet ["Public Internet Ingress"]
        ClientSPA["Client Browser (React 19 SPA)"]
        VaultAPI["Payment Gateway Vault (Accept.js)"]
    end

    subgraph AWSCloud ["AWS VPC (Production Region)"]
        ALB["AWS Application Load Balancer (ALB)<br/>TLS 1.3 Termination"]

        subgraph PrivateSubnet ["Private Isolated VPC Subnet"]
            subgraph FargateCluster ["AWS ECS Fargate Cluster"]
                APISvc["AdonisJS v6 API Service<br/>(Node 22 LTS / Pure ESM / Port 3333)"]
                WorkerSvc["Email & PDF Worker Microservice<br/>(BullMQ Worker / Port 3334)"]
            end

            subgraph StorageDataTier ["Data Tier & State Synchronization"]
                RedisEngine[("Redis 7 Alpine Cluster<br/>BullMQ & Permission Cache")]
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
    APISvc -->|"Lucid ORM / PgPool (Port 5432)"| RDSPG
    APISvc -->|"Multipart Put / Fetch Stream"| S3Storage
    WorkerSvc -->|"Read Invoice Binary Stream"| S3Storage
    
    APISvc -.->|"IAM Role Secret Fetch"| SecretsMgr
    WorkerSvc -.->|"IAM Role Secret Fetch"| SecretsMgr
