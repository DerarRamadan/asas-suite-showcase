<div align="center">

# 🏥 Asas Enterprise Suite

### Medical ERP · Biometric Attendance · Vault Management · Remittance Core

**A working enterprise in one polyglot monorepo — 16 packages, 7 vertical apps, zero cloud required.**

[![Private Codebase](https://img.shields.io/badge/STATUS-private_codebase_·_case_study-C85A32?style=for-the-badge)](https://derar.ly)
[![Portfolio](https://img.shields.io/badge/PORTFOLIO-derar.ly-1E1E1E?style=for-the-badge&logo=safari&logoColor=white)](https://derar.ly)
[![Email](https://img.shields.io/badge/CONTACT-walkthrough_on_request-6B655C?style=for-the-badge&logo=gmail&logoColor=white)](mailto:zzxxccz908@gmail.com)

<sub>Architected & led by [Derar Ramadan](https://github.com/DerarRamadan) — Head of Programming @ Ministry of Agriculture, Libya</sub>

</div>

---

## Why this exists

> *"Where I work, the connection drops — so the software can't."*

Libyan clinics, exchange houses, and warehouses cannot bet their operations on a datacenter that might be a 3,000 ms round-trip away — or unreachable entirely. **Asas Suite** is the answer: a family of enterprise applications that run **fully offline on the LAN**, ship as ultra-small native binaries, and speak **Arabic and English with full RTL/LTR parity** from the database up to the thermal printhead.

This repository is the **public case study**: architecture, scale, and engineering decisions. The production codebase is private (client confidentiality) — a guided walkthrough is available on request.

---

## Scale at a glance

| | |
|:---|:---|
| **07** | vertical applications — each shipping its own client, server, and desktop target |
| **08** | shared infrastructure packages every app stands on |
| **05** | languages — TypeScript · Rust · Python · SQL · PowerShell |
| **03** | targets per app — client · server · desktop |
| **< 50 MB** | zero-cloud Rust/Tauri binaries |

---

## System architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Desktop & Client Layer                                          │
│  ┌──────────────────────────────┐ ┌───────────────────────────┐ │
│  │ Tauri v2 Shell (Rust)        │ │ React 19 SPA              │ │
│  │ · Tokio UDP socket (:5757)   │ │ · Zustand · TanStack Query│ │
│  │ · WMI fingerprinting         │ │ · Tailwind v4 · Radix UI  │ │
│  │ · AES-256-GCM HWID licensing │ │ · Bilingual RTL / LTR     │ │
│  └──────────────────────────────┘ └───────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────────┘
              HTTP / WebSocket over LAN · UDP auto-discovery
┌───────────────────────────┴─────────────────────────────────────┐
│  Local-First Application Servers                                 │
│  ┌──────────────────────────────┐ ┌───────────────────────────┐ │
│  │ Fastify Engine (Bun/Node)    │ │ Drizzle ORM               │ │
│  │ · Route → Service → Repo     │ │ · SQLite (WAL) ⇄ Postgres │ │
│  │ · Zod v4 runtime validation  │ │ · WAL checkpoints         │ │
│  │ · RBAC · UDP responder       │ │ · Schema drift verify     │ │
│  └──────────────────────────────┘ └───────────────────────────┘ │
└───────────────────────────┬─────────────────────────────────────┘
                 Native bridges & secure tunnels
┌───────────────────────────┴─────────────────────────────────────┐
│  Infrastructure & Hardware                                       │
│  · @asas/printing — ESC/POS · Arabic glyph shaper · VFD pole    │
│  · ADMS / ZKTeco — biometric punch & template sync              │
│  · @asas/relay-hub — reverse-proxy WebSocket tunnel (NAT)       │
│  · @asas/backup — snapshot scheduler · encrypted off-site vault │
└─────────────────────────────────────────────────────────────────┘
```

**Zero-config networking:** clients find their server with a UDP broadcast on the LAN — answered with the exact IP and port. No hardcoded addresses, no IT tickets.

**Arabic on thermal paper:** every receipt printed in Arabic passes through a custom glyph shaper and 1-bit rasterizer before it reaches the printhead — because ESC/POS doesn't do Arabic shaping natively.

---

## The seven vertical applications

### 1. Sink Medical Services — Clinic ERP
Kills paperwork, lost patient histories, uncollected balances, and scheduling conflicts — with zero cloud downtime risk.
- EHR/EMR: visit histories, diagnoses, prescriptions, lab tests
- Multi-doctor scheduling with color-coded live queues
- Billing: insurance, service packages, doctor revenue shares, daily closures
- Thermal prescriptions & invoices with Arabic text shaping
- Offline-first LAN operation with automated daily snapshots

`Tauri v2 · React 19 · Fastify · Drizzle · Better-SQLite3 · pkg` — [public source snapshot →](https://github.com/DerarRamadan/sink-medical-services)

### 2. Fingerprint Manager — Workforce HR
Automates attendance across distributed ZKTeco devices — no proprietary cloud subscription required.
- ADMS push protocol + ZKTeco binary protocol ingestion
- Real-time punches + fingerprint & facial template sync
- Multi-branch hierarchy: departments, roles, shifts, flexible rota
- Leave balances, entitlements, mission orders, exit permissions
- Punch-log reconciliation into payroll-ready timesheets

`Fastify · ADMS parser · Python bridge · Drizzle · React 19 · Tauri v2`

### 3. Remittance Manager — Forex & Vault
Eliminates exchange accounting discrepancies, untracked broker settlements, and customer credit defaults.
- Multi-currency ledger: physical cash vaults + digital balances
- Strict FIFO credit/debt repayment engine per customer
- Statements of account — PDF export + thermal audit receipts
- Business-day lifecycle with evening variance detection
- Anti-fraud audit trail on every modification & cancellation

`Fastify · Drizzle · DDD (Money · Vault · Transaction · Debt · Card) · React 19`

### 4. Access Guard — Physical Security
Controls doors, barriers, and turnstiles with real-time biometric and RFID validation.
- ADMS-VL visible-light face recognition + RFID controllers
- Time-zone permissions by shift, day of week, clearance level
- Emergency lockdown triggers + evacuation muster rolls
- Entry/exit telemetry, visitor tracking, incident alerts

`Fastify · ADMS-VL listeners · Drizzle · React 19 · Tauri shell`

### 5. Asas Warehouse — Supply Chain
Ends inventory leakage, inaccurate stock counts, untracked tool handovers, and supplier mismatches.
- Multi-category SKU catalog + barcode scanning + low-stock alerts
- Employee custody: asset handover/return with digital sign-off
- Purchase orders, supplier ledgers, goods-receipt verification
- Blind vs. physical stocktaking with discrepancy reports
- WAC + FIFO inventory valuation reporting

`Fastify · Drizzle · Better-SQLite3/Postgres · React 19 · Radix · Tauri v2`

### 6. Oil Shop Manager — Automotive POS
Keeps up with quick-lube cashiers: bundle pricing, vehicle history, and batch expiration in one POS.
- Split payments: cash, card, deferred credit, customer accounts
- Dynamic service bundling with automatic discount calculation
- Batch allocator: lot numbers, oil viscosity, expiration dates
- Vehicle service history indexed by license plate
- WebSocket broadcast syncing every cashier terminal live

`Fastify · WebSockets · Drizzle (dual schema) · React 19 · Tauri v2`

### 7. Asas Database Server — Infrastructure GUI
Lets non-technical staff host, monitor, back up, and discover LAN database instances — one click.
- React + Tauri GUI over embedded PostgreSQL / SQLite services
- One-click start/stop, port binding, connection telemetry
- QR + UDP broadcast: clients auto-connect with zero IP config
- Background backup scheduler with point-in-time restore

`Tauri v2 · React 19 · QR generator · PostgreSQL process manager`

---

## The eight infrastructure packages

| Package | Responsibility |
|:---|:---|
| `@asas/printing` | Native printing engine: ESC/POS byte-streams to USB/serial, cash-drawer pulses, VFD pole drivers, Arabic glyph shaping + rasterization for thermal paper. |
| `@asas/relay-hub` | Cloud-hybrid NAT traversal: Fastify + WebSocket reverse proxy so on-prem servers behind CGNAT reach cloud/mobile clients without port forwarding. |
| `@asas/core` | Generic 3-tier base repositories, service orchestrators, crypto utilities, Pino structured logging, UDP discovery emitters. |
| `@asas/types` | Monorepo-wide domain models, DTOs, API payloads, and RPC contracts — type safety across every microservice. |
| `@asas/db` | Universal data access: connection pooling, Drizzle helpers, dynamic SQLite ⇄ PostgreSQL dialect adapters. |
| `@asas/migrations` | Unified migration runner with automated pre-push schema drift detection (`db:verify`). |
| `@asas/ui` | Design system: tokens, Radix accessible primitives, Lucide icons, Sonner toasts — full RTL Arabic/English. |
| `@asas/backup` | Disaster recovery vault: snapshotting, WAL checkpoints, compression, encrypted Google Drive uploads. |

---

## Technology matrix

| Layer | Stack |
|:---|:---|
| Languages | TypeScript · Rust · SQL · Python · PowerShell |
| Desktop shell | Tauri v2 — Tokio · WMI · winreg |
| Frontend | React 19 · Vite · Tailwind v4 · Radix · Zustand |
| Backend | Fastify 5 · Bun / Node.js · Zod v4 |
| Databases | SQLite (WAL) ⇄ PostgreSQL — dual dialect |
| IoT & protocols | ESC/POS · ADMS/ZKTeco · VFD serial · UDP broadcast |
| Security | AES-256-GCM licensing · SHA-256 HWIDv3 · JWT · audit trails |
| i18n | Arabic (RTL) / English (LTR) — full parity |

---

## License & code availability

This showcase repository (documentation, diagrams, case-study text) is released under [MIT](./LICENSE).

The **Asas Suite production codebase remains private** — it contains client data models and is deployed in live institutions. A guided code walkthrough is available to serious parties: **zzxxccz908@gmail.com**.

<div align="center">

<sub><i>© 2026 Derar Ramadan · [derar.ly](https://derar.ly) · Tripoli, Libya / Remote Worldwide</i></sub>

</div>
