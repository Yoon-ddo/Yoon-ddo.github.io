---
title: Project1:Foreign Currency Asset Management System Development
toc: true
toc_sticky: true
toc_label: "Contents of Page"
categories:
  - Study
tags:
  - MSA_architecture
---

<br><br><br>

# Foreign Currency Asset Management System

## Overview

A backend-focused financial platform for managing multi-currency assets, exchange rates, portfolio valuation, and AI-powered financial summaries.

This project focuses on:

- High-volume financial transaction processing
- Exchange-rate integration
- Wallet ledger consistency
- Portfolio valuation
- Event-driven microservice architecture
- Reliable settlement and transaction tracking

---

# 1. Core Features

## 1.1 Must-Have Features (MVP)

### 1) Multi-Currency Wallet Management

Manage balances for multiple foreign currencies per user.

#### Key Points

- Single-user MVP design (multi-user architecture supported)
- ISO currency codes (`USD`, `JPY`, `EUR`, etc.)

#### Features

- Create currency wallets
- Currency exchange
- Balance inquiry
- Deposit / withdrawal
- Wallet balance updates
- Transaction memo support
- Balance validation
- Insufficient balance prevention

#### Main Entities

- `Wallet`
- `WalletTransaction`

---

### 2) Exchange Rate API Integration (Korea Eximbank)

Integrates external exchange-rate APIs from Korea Eximbank.

#### Key Points

- External API → Internal DTO conversion
- Exchange-rate caching required

#### Features

- Retrieve latest exchange rates
- Store currency exchange rates
- Exception handling for API failures
- Snapshot fallback strategy

---

### 3) Portfolio Valuation (KRW / USD)

Convert and aggregate assets based on a selected base currency.

#### Key Points

- Accurate financial calculations
- Decimal precision handling

#### Features

- Base currency selection (`KRW` / `USD`)
- Total portfolio valuation
- Per-currency converted values

---

## 1.2 Should-Have Features

### 1) Exchange Rate Snapshot History

Store historical exchange-rate snapshots.

#### Features

- Daily exchange-rate snapshots
- Historical exchange-rate lookup
- Duplicate snapshot prevention

#### Main Entity

- `ExchangeRateSnapshot`

---

### 2) Portfolio Profit & Loss (P/L)

Track valuation-based profit and loss.

#### Features

- Portfolio valuation comparison
- Currency-level P/L
- Total portfolio P/L
- Percentage change calculation

#### Notes

- Unrealized valuation P/L only
- No realized profit calculation

---

## 1.3 Could-Have Features

### AI Summary Reports (LLM-based)

Generate AI-powered portfolio and exchange-rate summaries.

#### Input Data

- 7-day / 14-day exchange-rate snapshots
- Currency fluctuation rates
- Portfolio allocation ratio

#### Features

- Exchange-rate trend summaries
- Market factor explanations
- Portfolio impact analysis
- Non-predictive financial commentary

#### Example

> “USD/KRW increased by 1.8% over the last 7 days.”

---

# 2. Development Schedule

## Project Timeline

| Period | Duration |
|---|---|
| 2026-01-26 ~ 2026-02-23 | 4 Weeks |

### Weekly Goals

| Week | Goal |
|---|---|
| Week 1 | Architecture & environment setup |
| Week 2 | MVP implementation |
| Week 3 | Exchange-rate history & P/L |
| Week 4 | AI reporting & system refinement |

---

# 3. Technology Stack

## Backend

- Java 17
- Spring Boot
- Spring Data JPA (Hibernate)
- MySQL 8
- Flyway

## Validation & Documentation

- Jakarta Validation
- OpenAPI (Swagger)

## Testing

- JUnit5
- Spring Boot Test

## Infrastructure

- Docker
- Docker Compose
- Kafka
- Kubernetes

---

# 4. System Architecture

## Architecture Goals

- Event-driven microservice architecture
- High scalability
- Reliable transaction processing
- Clear domain separation
- Financial data consistency

---

## Service Structure

```text
services/
├── user-service
├── wallet-service
├── exchange-rate-service
├── valuation-service
└── report-service
```

---

## Shared Libraries

```text
libs/
├── common-web
├── common-event
└── common-kafka
```

---

## Infrastructure

```text
infra/
└── k8s/
```

Includes:

- Kubernetes deployment manifests
- HPA configuration
- CronJobs
- ConfigMap / Secret management

---

# 5. Database Design

## users

Stores user account information.

| Column | Description |
|---|---|
| id | User identifier |
| email | Login email |
| name | Display name |
| created_date | Created timestamp |
| updated_date | Updated timestamp |

---

## wallets

Stores currency wallet balances.

| Column | Description |
|---|---|
| user_id | Owner user |
| currency_code | ISO currency code |
| balance | Current balance |

---

## wallet_events

Represents business-level wallet actions.

Examples:

- Deposit
- Withdrawal
- Exchange

---

## wallet_transactions

Ledger table for wallet balance changes.

### Features

- Tracks all financial balance changes
- Supports auditing and validation
- Stores post-transaction balance snapshots

---

## exchange_rate_snapshots

Stores daily exchange-rate snapshot metadata.

---

## exchange_rate_snapshot_details

Stores per-currency exchange-rate data.

---

## ai_reports

Stores AI-generated financial summaries.

---

# 6. Transaction Ledger Design

## Wallet Event Structure

### wallet_events

Represents a single business action:

- Deposit
- Withdrawal
- Exchange

### wallet_transactions

Represents actual ledger entries.

#### Example

| Action | Transactions |
|---|---|
| Deposit | 1 CREDIT |
| Withdrawal | 1 DEBIT |
| Exchange | 1 DEBIT + 1 CREDIT |

---

# 7. Event-Driven Architecture

Kafka-based asynchronous communication between services.

## Event Examples

```text
WalletEventCreated
ExchangeRateSnapshotCreated
```

## Benefits

- Loose coupling
- Scalability
- Reliable event processing
- Better service isolation

---

# 8. Reliability & Financial Consistency

## Key Considerations

- Idempotency handling
- Transaction consistency
- Exception handling
- Ledger traceability
- Financial precision
- High-volume batch processing support

---

# 9. Future Improvements

- Real-time exchange-rate streaming
- Multi-user authentication
- Distributed transaction handling
- Redis caching
- CI/CD pipeline
- Observability & monitoring
- Multi-region deployment
