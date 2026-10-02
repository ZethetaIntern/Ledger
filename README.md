# 💰 BED-6C-DevanshNegi-Ledger

> **Production-grade ledger system with double-entry accounting, multi-currency support, concurrency control, immutable audit trails, and financial reporting.**

<div align="center">

**Double-Entry Accounting • Multi-Currency FX • Immutable Ledger • Audit Integrity • Concurrency Safety**

</div>

---

## 🛠️ Tech Stack

<div align="center">

![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-API-009688?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-2.0-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white)
![Alembic](https://img.shields.io/badge/Alembic-Migrations-499848?style=flat-square)
![Pydantic](https://img.shields.io/badge/Pydantic-v2-E92063?style=flat-square)
![APScheduler](https://img.shields.io/badge/APScheduler-Scheduling-2C3E50?style=flat-square)
![JWT](https://img.shields.io/badge/JWT-Authentication-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-Optional-DC382D?style=flat-square&logo=redis&logoColor=white)
![Pytest](https://img.shields.io/badge/Pytest-Testing-0A9EDC?style=flat-square&logo=pytest&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?style=flat-square&logo=docker&logoColor=white)

</div>

---

# 📌 Overview

**BED-6C-DevanshNegi-Ledger** is a ledger and accounting backend built according to the **Zetheta BED-6C project specification**.

The system implements a financial ledger around:

- 📒 Double-entry accounting
- 💱 Multi-currency transactions
- 🔐 Immutable ledger entries
- 🔗 Hash-chain audit integrity
- 🔁 Idempotent state-changing operations
- 🔒 Concurrency-safe account updates
- ↩️ Reversals and refunds
- 📊 Financial reporting
- 🧾 Account statements
- 🔍 Audit verification
- 🗂️ PostgreSQL partitioning
- 🔄 Zero-downtime migration support

**Project status:** All 15 planned days are complete according to the project specification, including all 20 transaction types, FX support, concurrency controls, reversals/refunds, audit verification, reporting, partitioning, and zero-downtime migrations.

For the detailed self-assessment and deferred items, see:

```text
docs/reviews/final-review.md
```

---

# 🧠 Accounting Model

The system is built around the fundamental double-entry accounting invariant:

```text
For every journal:

        SUM(DEBITS)
             =
        SUM(CREDITS)

        ↓

For EACH currency independently
```

The system intentionally does **not** mix different currencies into a single global debit/credit total.

```text id="j9m2o5"
             Journal Entry
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
      USD                   EUR
        │                   │
   ┌────┴────┐         ┌────┴────┐
   │         │         │         │
 Debit     Credit    Debit     Credit
   │         │         │         │
   └────┬────┘         └────┬────┘
        │                   │
        ▼                   ▼
      Balanced            Balanced
```

This preserves accounting correctness for multi-currency journals.

---

# 🏗️ System Architecture

```text id="r2i1k7"
                         ┌──────────────────────┐
                         │       Clients        │
                         │ Web / API / Services │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       FastAPI        │
                         │      Controllers     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      Middleware      │
                         │ Auth / Idempotency   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       Services       │
                         │                      │
                         │ • Transactions       │
                         │ • Transfers          │
                         │ • FX                 │
                         │ • Reversals         │
                         │ • Refunds             │
                         │ • Reporting          │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     SQLAlchemy       │
                         │        Models        │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │     PostgreSQL 15    │
                         │                      │
                         │ • Ledger Tables      │
                         │ • Triggers            │
                         │ • Partitions          │
                         │ • Advisory Locks     │
                         └──────────┬───────────┘
                                    │
                ┌───────────────────┼───────────────────┐
                │                   │                   │
                ▼                   ▼                   ▼
           Audit Chain          Reports             Jobs
                │                   │                   │
                ▼                   ▼                   ▼
             Hashes           Statements        APScheduler
```

---

# 🔄 Ledger Transaction Flow

```text id="rj9a2q"
Transaction Request
        │
        ▼
Authentication
        │
        ▼
Idempotency Check
        │
        ▼
Validate Request
        │
        ▼
Acquire Account Locks
        │
        ▼
Create Journal
        │
        ▼
Create Ledger Entries
        │
        ▼
Verify Double-Entry Balance
        │
        ▼
Generate Hash
        │
        ▼
Commit Transaction
        │
        ▼
Return Result
```

The system performs the idempotency check **before business logic executes**, preventing duplicate state-changing requests from progressing into transaction processing.

---

# 💳 Supported Transaction Types

The ledger supports all **20 transaction types** defined by the project specification.

```text id="0j1y7j"
┌─────────────────────────────────────────┐
│          Transaction Categories         │
├─────────────────────────────────────────┤
│                                         │
│ Deposit                                 │
│ Card Deposit                            │
│ Withdrawal                              │
│ Transfer                                │
│ Merchant Payment - QR                   │
│ Merchant Payment - Online               │
│ Bill Payment                            │
│ Interest Accrual                        │
│ Interest Payout                         │
│ Fee Deduction                           │
│ Cashback Credit                         │
│ Promotional Credit                      │
│ Loan Disbursement                       │
│ Loan EMI                                │
│ FX Transaction                          │
│ Reversal                                │
│ Refund                                  │
│ Chargeback                              │
│ Reward Redemption                       │
│ Account Closure                         │
│                                         │
└─────────────────────────────────────────┘
```

---

# 💱 Multi-Currency FX

The ledger supports transactions involving multiple currencies.

FX processing is designed around the accounting rule that debit and credit equality must be maintained **per currency**.

```text id="yq2g9j"
             FX Transaction
                   │
          ┌────────┴────────┐
          │                 │
          ▼                 ▼
        Source            Target
       Currency           Currency
          │                 │
          ▼                 ▼
      Source Entry      Target Entry
          │                 │
          └────────┬────────┘
                   ▼
             FX Journal
                   │
                   ▼
          Per-Currency Balance
```

Sample FX rate data is seeded during application startup.

---

# 🔐 Immutable Ledger

Ledger entries are designed to be immutable.

Once posted:

```text id="z7qj0v"
POSTED
   │
   │
   ├───────────────┐
   │               │
   ▼               ▼
No UPDATE        No DELETE
   │
   │
   ▼
Only narrow
POSTED → REVERSED
transition
```

PostgreSQL triggers enforce the immutability rules independently of application code.

This means that even if an application-level safeguard is bypassed, the database continues to enforce the ledger's integrity constraints.

---

# 🔗 Hash-Chain Audit Trail

Every ledger entry participates in a hash chain.

Conceptually:

```text id="e0ddj6"
Entry 1
   │
   ▼
SHA256(
  entry_fields
  +
  previous_hash
)
   │
   ▼
Entry 2
   │
   ▼
SHA256(
  entry_fields
  +
  hash_1
)
   │
   ▼
Entry 3
   │
   ▼
SHA256(
  entry_fields
  +
  hash_2
)
   │
   ▼
...
```

Each entry contains:

```text id="bkg0aa"
hash = SHA256(entry_fields + previous_hash)
```

The result is a single global chain.

---

# 🔍 Audit Verification

The API provides:

```http id="l6x3bn"
GET /api/v1/audit/verify
```

The verification process re-walks the chain and re-derives every hash.

```text id="f6m2qn"
Stored Ledger
     │
     ▼
Read Entry
     │
     ▼
Recalculate Hash
     │
     ▼
Compare
     │
 ┌───┴────┐
 │        │
Match   Mismatch
 │        │
 ▼        ▼
Continue Exact Break Point
```

If tampering is detected, the verification process can identify the exact point where the chain breaks.

Additional endpoint:

```http id="m8zj3c"
GET /api/v1/audit/hash-chain
```

---

# 🔁 Idempotency

Every state-changing request requires an idempotency key.

```text id="f4p6xj"
Client Request
      │
      ▼
Idempotency Key
      │
      ▼
Already Processed?
      │
 ┌────┴────┐
 │         │
Yes        No
 │         │
 ▼         ▼
Return   Execute
Existing Business
Result    Logic
            │
            ▼
       Persist Result
```

The key is checked before business logic is executed.

This protects financial operations from duplicate requests caused by:

- Client retries
- Network timeouts
- Application retries
- Duplicate submissions

---

# 🔒 Concurrency Control

Financial ledger systems must protect account balances when multiple transactions occur concurrently.

The project uses:

- `SELECT FOR UPDATE`
- Deterministic multi-account lock ordering
- PostgreSQL transactions
- Real-thread concurrency/load tests

```text id="x1f9e6"
Transaction A ─────┐
                   │
                   ▼
              Account Lock
                   │
                   ▼
              Update Balance
                   │
                   ▼
                 Commit
                   │
                   ▼
              Release Lock
                   │
                   ▼
Transaction B ─────┘
```

Deterministic lock ordering helps reduce the risk of deadlocks when multiple accounts are involved.

The concurrency strategy is tested against a real PostgreSQL instance rather than relying only on mocks.

---

# 💰 Money Representation

The system deliberately avoids floating-point arithmetic for financial values.

### Database

```text
NUMERIC(19,4)
```

### Application

```text
Python Decimal
```

### Entity IDs

```text
UUID v7
```

UUID v7 provides time-sortable identifiers for persisted entities.

```text
Money
  ↓
NUMERIC(19,4)
  ↓
Python Decimal
  ↓
No binary floating-point calculations
```

---

# ↩️ Reversals & Refunds

The ledger supports:

- Reversals
- Refunds
- Chargebacks

These operations do not require rewriting historical posted ledger entries.

Instead, the accounting model preserves the original record and represents the compensating financial event separately.

```text id="b7cn3e"
Original Transaction
        │
        ▼
     POSTED
        │
        ▼
Reversal / Refund
        │
        ▼
Compensating Entry
        │
        ▼
Auditable History
```

This preserves the historical ledger trail.

---

# 📊 Reporting Suite

The system provides several accounting reports.

### Trial Balance

```http
GET /api/v1/trial-balance
```

### Account Statement

```http
GET /api/v1/accounts/{code}/statement
```

The account statement also supports CSV export.

### Income Statement

```http
GET /api/v1/reports/income-statement
```

### Balance Sheet

```http
GET /api/v1/reports/balance-sheet
```

### Currency Exposure

```http
GET /api/v1/reports/currency-exposure
```

---

# 📈 Reporting Architecture

```text id="a8mtjv"
                    Ledger Entries
                          │
                          ▼
                 ┌─────────────────┐
                 │ Reporting Layer │
                 └────────┬────────┘
                          │
          ┌───────────────┼────────────────┐
          │               │                │
          ▼               ▼                ▼
    Trial Balance    Account Statement   Financial Reports
                                          │
                              ┌───────────┼───────────┐
                              │           │           │
                              ▼           ▼           ▼
                         Income Sheet  Balance    Currency
                                      Sheet       Exposure
```

---

# 🗃️ PostgreSQL Partitioning

The project includes PostgreSQL-specific partitioning support.

```text id="5w1u8c"
                 ledger_entries
                       │
                       ▼
                Partition Strategy
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
      Partition A  Partition B  Partition C
```

Partition management utilities are available under:

```text
scripts/manage_partitions.py
scripts/archive_partition.py
```

Partitioning is tested through the integration/load suite against PostgreSQL because the relevant behavior depends on database-specific functionality.

---

# 🔄 Zero-Downtime Migrations

The project includes migration support designed around zero-downtime deployment requirements.

Database changes are managed through:

```text
Alembic
   │
   ▼
8 Migration Versions
   │
   ▼
Schema Evolution
```

Migration files are located under:

```text
migrations/versions/
```

---

# 🌱 Seed Data

The project includes seed data for:

### Chart of Accounts

```text
20 Accounts
```

### FX Rates

Sample foreign-exchange rate data is included for development and testing.

Seed scripts are located under:

```text
seeds/
```

---

# 📁 Repository Structure

The repository mirrors the specification's mandatory layout.

```text id="j7x1c4"
BED-6C-DevanshNegi-Ledger/
│
├── src/
│   ├── config/
│   ├── models/
│   ├── services/
│   ├── controllers/
│   ├── middleware/
│   ├── utils/
│   ├── validators/
│   ├── routes/
│   └── jobs/
│
├── migrations/
│   └── versions/
│       └── 8 Alembic migrations
│
├── seeds/
│   ├── Chart of Accounts
│   └── FX rate seed data
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── load/
│
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── schema/
│   ├── incident-responses/
│   └── reviews/
│
├── scripts/
│   ├── seed.py
│   ├── verify_hashes.py
│   ├── manage_partitions.py
│   └── archive_partition.py
│
├── requirements.txt
├── .env.example
└── docker-compose.yml
```

---

# 🔌 API Overview

The application exposes **28 endpoints** covering transaction processing, audit verification, and financial reporting.

## 💳 Transactions

| Endpoint | Purpose |
|---|---|
| `/deposit` | Deposit transaction |
| `/deposit-card` | Card deposit |
| `/withdraw` | Withdrawal |
| `/transfer` | Account transfer |
| `/merchant-payment/qr` | QR merchant payment |
| `/merchant-payment/online` | Online merchant payment |
| `/bill-payment` | Bill payment |
| `/interest-accrual` | Interest accrual |
| `/interest-payout` | Interest payout |
| `/fee-deduction` | Fee deduction |
| `/cashback-credit` | Cashback credit |
| `/promotional-credit` | Promotional credit |
| `/loan-disbursement` | Loan disbursement |
| `/loan-emi` | Loan EMI |
| `/fx` | FX transaction |
| `/reversal` | Transaction reversal |
| `/refund` | Refund |
| `/chargeback` | Chargeback |
| `/reward-redemption` | Reward redemption |
| `/account-closure` | Account closure |

---

## 🔍 Audit

| Endpoint | Purpose |
|---|---|
| `/audit/verify` | Verify hash-chain integrity |
| `/audit/hash-chain` | Inspect hash-chain data |

---

## 📊 Reports

| Endpoint | Purpose |
|---|---|
| `/trial-balance` | Generate trial balance |
| `/accounts/{code}/statement` | Account statement |
| `/reports/income-statement` | Income statement |
| `/reports/balance-sheet` | Balance sheet |
| `/reports/currency-exposure` | Currency exposure |

The complete API definition is available at:

```text
docs/api/openapi.yaml
```

The OpenAPI specification is generated from the live application.

---

# ❤️ Health Check

After starting the application:

```http
GET http://localhost:8000/api/v1/health
```

Swagger UI:

```text
http://localhost:8000/docs
```

---

# 🚀 Quick Start

## 1. Configure Environment

```bash
cp .env.example .env
```

Update the environment variables as required.

---

## 2. Start the Application

```bash
docker compose up --build
```

This starts:

- PostgreSQL 15
- Redis
- Database migrations
- Seed data
- FastAPI application

---

## 3. Database Initialization

Startup performs:

```text id="pg4k3u"
PostgreSQL 15
      │
      ▼
8 Alembic Migrations
      │
      ▼
Schema + Triggers + Partitions
      │
      ▼
Chart of Accounts
      │
      ▼
Sample FX Rates
      │
      ▼
FastAPI Application
```

The Chart of Accounts contains **20 seeded accounts**.

---

# 🧪 Testing

The project contains **32 tests**:

```text
6 Unit Tests
+
26 Integration / Load Tests
=
32 Tests
```

Run through Docker:

```bash
docker compose --profile test run tests
```

Or locally:

```bash
pip install -r requirements.txt

pytest --cov=src --cov-report=term-missing
```

---

# 🧪 Test Architecture

```text id="j3t0v1"
                    Test Suite
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
       Unit Tests             Integration
       6 tests                + Load Tests
                                  │
                                  ▼
                           PostgreSQL 15
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
                   ▼              ▼              ▼
                Triggers      Advisory Locks  Partitions
```

The integration/load suite requires real PostgreSQL because several important behaviors are PostgreSQL-specific and are not meaningfully reproduced using SQLite.

---

# 🔍 Hash Verification Utility

The repository includes:

```text
scripts/verify_hashes.py
```

This provides an additional operational utility for verifying ledger hash-chain integrity.

The API-level verification is also exposed through:

```http
GET /api/v1/audit/verify
```

---

# 📚 Documentation

The repository contains extensive supporting documentation.

| Directory / File | Purpose |
|---|---|
| `docs/architecture/` | Four Architecture Decision Records |
| `docs/incident-responses/` | Six incident post-mortems |
| `docs/reviews/` | Phase reviews and final retrospective |
| `docs/submission-notes.md` | Specification corrections and reasoning |
| `docs/api/openapi.yaml` | Auto-generated OpenAPI 3.0 specification |

---

# 🏛️ Architecture Decision Records

The architecture documentation contains **4 ADRs** covering areas such as:

- Technology stack
- Concurrency
- Data retention
- Database migrations

These decisions document not only what was implemented, but why particular approaches were selected.

---

# 🚨 Incident Documentation

The repository contains **6 post-mortems**, with one documented response for each incident card.

These provide a record of:

```text
Incident
   │
   ▼
Investigation
   │
   ▼
Root Cause
   │
   ▼
Resolution
   │
   ▼
Preventive Measures
```

---

# 📝 Submission Notes

The project includes:

```text
docs/submission-notes.md
```

This document records:

- Deliberate specification errors that were identified
- Corrections made to the accounting model
- Reasoning behind those corrections
- Known limitations
- Implementation decisions

The accounting corrections were derived by tracing the specification's worked examples through to balance-check failures rather than simply implementing the specification text without verification.

---

# 🧠 Core Engineering Principles

## 1. Double-Entry Integrity

Every journal must satisfy:

```text
SUM(Debits) = SUM(Credits)
```

for each currency independently.

---

## 2. Immutable Financial History

Posted ledger entries cannot simply be modified or deleted.

The database itself enforces immutability.

---

## 3. Cryptographic Auditability

Every ledger entry participates in a SHA-256 hash chain.

Tampering can therefore be detected through chain verification.

---

## 4. Idempotent Financial Operations

State-changing operations require idempotency keys.

This protects against duplicate financial transactions.

---

## 5. Concurrency Safety

Account-level operations use PostgreSQL row locking and deterministic lock ordering.

---

## 6. Exact Financial Arithmetic

Money is represented using:

```text
PostgreSQL → NUMERIC(19,4)
Python     → Decimal
```

No floating-point money calculations are used.

---

## 7. Database-Enforced Invariants

Critical financial invariants are not left exclusively to application code.

PostgreSQL triggers enforce important ledger integrity rules at the database layer.

---

# 📈 Engineering Highlights

```text id="x1ajf9"
┌───────────────────────────────────────────────┐
│              FINANCIAL LEDGER                 │
├───────────────────────────────────────────────┤
│                                               │
│  Double-Entry Accounting                      │
│           ↓                                   │
│  Multi-Currency FX                            │
│           ↓                                   │
│  Idempotent Transactions                      │
│           ↓                                   │
│  PostgreSQL Concurrency Control               │
│           ↓                                   │
│  Immutable Ledger Entries                     │
│           ↓                                   │
│  SHA-256 Hash Chain                           │
│           ↓                                   │
│  Reversals / Refunds / Chargebacks            │
│           ↓                                   │
│  Financial Reporting                          │
│           ↓                                   │
│  Partitioned Storage                          │
│           ↓                                   │
│  Zero-Downtime Migrations                     │
│                                               │
└───────────────────────────────────────────────┘
```

### Key Engineering Concepts Demonstrated

- 💰 Double-entry accounting
- 💱 Multi-currency accounting
- 🔐 Immutable data design
- 🔗 Cryptographic hash chains
- 🔁 Idempotency
- 🔒 PostgreSQL row locking
- 🧵 Concurrent transaction processing
- ↩️ Reversals and refunds
- 📊 Financial reporting
- 🗂️ PostgreSQL partitioning
- 🔄 Alembic migrations
- 🐳 Dockerized infrastructure
- 🧪 Unit, integration, and load testing
- 📚 Architecture Decision Records
- 🚨 Incident-response documentation

---

# ⚠️ Honest Project Status

The repository states that the full 15-day implementation is complete, including:

```text
20 Transaction Types
        +
Multi-Currency FX
        +
Concurrency Control
        +
Reversals / Refunds
        +
Audit Verification
        +
Reporting Suite
        +
Partitioning
        +
Zero-Downtime Migrations
        =
Completed Specification Scope
```

Deferred items and limitations are documented separately in:

```text
docs/reviews/final-review.md
docs/submission-notes.md
```

---

# 🤖 Acknowledgement of AI Assistance

This project was developed with assistance from **Claude (Anthropic, Sonnet 4.6)** for:

- Repository scaffolding
- SQLAlchemy model and migration authoring
- Hash-chaining utilities
- Concurrency-control utilities
- Service-layer implementation
- Test generation

The project documentation states that the major schema decisions, accounting-model corrections, and concurrency strategy were independently derived by tracing the specification's worked examples and investigating resulting balance-check failures.

Detailed reasoning is documented within the relevant service modules and ADRs.

---

# 👨‍💻 Author

<div align="center">

### Devansh Negi

**Backend / AI Engineer**

Python • FastAPI • PostgreSQL • Distributed Systems • AI/ML • Docker

[![GitHub](https://img.shields.io/badge/GitHub-devanshnegi88-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/devanshnegi88)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Devansh%20Negi-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/devansh-negi005)

</div>

---

<div align="center">

## 💰 BED-6C Ledger System

**Double-entry accounting with immutable auditability, concurrency safety, and multi-currency financial reporting.**

</div>
