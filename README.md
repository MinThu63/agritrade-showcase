<div align="center">

<img src="docs/logo.svg" alt="AgriTrade logo" width="96" />

# AgriTrade SME

**A ledger-first app for Myanmar commodity traders: sell, buy, and the maths does itself.**

[![Open the live app](https://img.shields.io/badge/%F0%9F%94%97%20Live%20app-open%20now-2f6f73?style=for-the-badge)](https://agritrade.kmtlabs.workers.dev)

<sub>👉 <a href="https://agritrade.kmtlabs.workers.dev">agritrade.kmtlabs.workers.dev</a> · sign-up needs admin approval</sub>

<br />

<a href="https://agritrade.kmtlabs.workers.dev"><img src="docs/screenshots/homepage.webp" alt="AgriTrade landing page" width="900" /></a>

<sub>The public landing page. Click to open the live app.</sub>

![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.7-3178C6?logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-8-646CFF?logo=vite&logoColor=white)
![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-F38020?logo=cloudflare&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-RLS-4169E1?logo=postgresql&logoColor=white)
![PWA](https://img.shields.io/badge/PWA-installable-5A0FC8)
![Tests](https://img.shields.io/badge/tests-1%2C077%20passing-2EA44F)

<br />

> 📌 **This is a showcase repository.** It contains the write-up and screenshots only.
> The source code is private.

</div>

---

## Table of Contents

- [Motivation](#motivation)
- [Features](#features)
- [Architecture](#architecture)
- [How It's Built](#how-its-built)
- [Technical Highlights](#technical-highlights)
- [Try It](#try-it)
- [Screenshots](#screenshots)
- [Roadmap](#roadmap)
- [License](#license)

---

## Motivation

- Small agri traders still keep paper ledgers and hand-held calculators
- Beans, sesame, pulses: many items, many prices, transport on top
- Mistakes in the maths cost real money
- Users are **not technical**, often on a phone, often Burmese-first

**AgriTrade turns one basket into one receipt, one ledger, and live stock.**

Units are **viss**. Money is **MMK**.

---

## Features

| | |
|---|---|
| 🧮 **Basket Calculator**<br/>Multi-item sale or purchase, transport per viss, live totals, one printable receipt | 📒 **Ledger Book**<br/>Every sale and purchase in one dated book. Period views, search, filters, CSV export, bulk entry |
| 📦 **Automatic Inventory**<br/>Stock moves with every trade. Low-stock and out-of-stock alerts, viss-per-bag view | 📊 **Insights**<br/>Money flow by month, stock value, sales and transport per commodity |
| 🧾 **Receipts**<br/>Grouped receipt with customer and car number. Reprint any old record | 👥 **Owner and Cashier roles**<br/>Cashiers join with a shop code. The owner approves them |
| 🛡️ **Admin approval**<br/>New owners wait for the platform admin to approve or decline | 🔐 **Safe sign-in**<br/>Myanmar phone checks, easy passwords refused, lockout after repeated failures |
| 🌐 **Burmese and English**<br/>One tap to switch. Light, dark and system themes | 📱 **Installable PWA**<br/>Works on phones. Tables turn into cards on small screens |

---

## Architecture

One Cloudflare Worker serves both the API and the app.

```mermaid
flowchart LR
    U["Trader's phone or laptop<br/>React PWA"] -->|"HTTPS + bearer token"| W

    subgraph CF["Cloudflare"]
        W["Worker<br/>REST API + static app"]
        H["Hyperdrive<br/>database connector"]
        R[("R2 bucket<br/>business logos")]
    end

    W --> H --> DB[("PostgreSQL<br/>one business_id on every table<br/>row-level security")]
    W --> R
```

Every request runs the same three gates before any business logic:

```mermaid
flowchart LR
    A["Request"] --> B["1. Valid token?"]
    B --> C["2. Which business?<br/>(from the session, never the request)"]
    C --> D["3. Allowed to do this?<br/>(role permission)"]
    D --> E["Module service"]
    E --> F[("PostgreSQL<br/>scoped to that business")]
```

---

## How It's Built

A monorepo with two apps and a modular backend.

```text
agritrade/
├── apps/web/     # React 19 PWA (Vite, TypeScript, React Router)
└── apps/api/     # One Worker, many modules, one database
    ├── auth        # sign-in, lockout, tokens, registration, approvals
    ├── business    # profile, team, shop code, logo
    ├── ledger      # entries, running balance, edit/delete, bulk, CSV
    ├── inventory   # stock, thresholds, bag sizes
    ├── calculator  # basket totals, grouped save
    ├── receipts    # receipt payloads
    ├── insights    # money flow and per-commodity numbers
    └── admin       # approvals, suspend, announcements, audit log
```

- 23 database migrations, applied in order
- Secrets live in the platform, never in the repo

---

## Technical Highlights

- **One money formula.** The server owns every total. The Calculator preview and the saved ledger use the same code, so they always agree
- **No overselling.** A sale that needs more stock than the shelf holds is refused, with a plain message
- **Multi-tenant by design.** `business_id` on every table, plus row-level security as a second lock
- **Safe retries.** Saves are idempotent, so a double tap or a flaky network does not double-book a sale
- **Workers-ready.** A fresh database pool per request, closed after the response. A shared pool hung in production
- **Careful auth.** PBKDF2-SHA-256, 15-minute lockout after 5 failed attempts, short-lived access tokens with revocable refresh tokens
- **Admin without snooping.** The admin console shows account details and counts only, never ledger or stock rows
- **Same rules on both sides.** The form and the server share one set of field checks, so the server never trusts the browser
- **Tested.** 687 API tests and 390 web tests, including property-based tests for money maths and lockout
- **Pure view-models.** Screen logic lives in small pure functions that are easy to test without a browser

---

## Try It

The app is live. Sign-up needs admin approval.

- 🔗 **Live app:** <https://agritrade.kmtlabs.workers.dev>
- Want a look inside? Create an account, or message me for a demo login

Source code is private, so there is nothing to clone or install here.

---

## Screenshots

<table>
  <tr>
    <td width="50%"><a href="docs/screenshots/dashboard.webp"><img src="docs/screenshots/dashboard.webp" alt="Dashboard" /></a><br /><sub><b>Dashboard</b>: running balance, stock value, today's sales and a "Buy soon" list at a glance</sub></td>
    <td width="50%"><a href="docs/screenshots/calculator.webp"><img src="docs/screenshots/calculator.webp" alt="Calculator" /></a><br /><sub><b>Calculator</b>: tap items to build a Sale or Purchase bill, live totals, one printable receipt</sub></td>
  </tr>
  <tr>
    <td width="50%"><a href="docs/screenshots/ledger.webp"><img src="docs/screenshots/ledger.webp" alt="Ledger Book" /></a><br /><sub><b>Ledger Book</b>: every trade in one dated book, with period views, search, CSV download and receipt reprint</sub></td>
    <td width="50%"><a href="docs/screenshots/inventory.webp"><img src="docs/screenshots/inventory.webp" alt="Inventory" /></a><br /><sub><b>Inventory</b>: stock health, stock value, top sellers, and a low-stock level for each item</sub></td>
  </tr>
  <tr>
    <td width="50%"><a href="docs/screenshots/insights.webp"><img src="docs/screenshots/insights.webp" alt="Insights" /></a><br /><sub><b>Insights</b>: money flow by month, and bought, sold, revenue and net for each commodity</sub></td>
    <td width="50%"><a href="docs/screenshots/profile.webp"><img src="docs/screenshots/profile.webp" alt="My Profile" /></a><br /><sub><b>My Profile</b>: business details, a hidden shop code for cashiers, and the team list</sub></td>
  </tr>
</table>

<sub>Screenshots come from a test account with sample data, so some figures are not realistic. Phone numbers are blurred. Click any image to enlarge.</sub>

---

## Roadmap

Ideas, not promises.

- [x] Calculator, Ledger Book, Inventory, Insights, receipts
- [x] Owner and Cashier roles with shop code
- [x] Low-stock alerts and viss-per-bag view
- [x] Admin console and owner approval
- [x] Burmese and English
- [ ] Marketplace: listings and inbox (shown as "coming soon" in the app)
- [ ] Stock history for each item
- [ ] Stock counts (cycle counts)
- [ ] Batches and stock aging
- [ ] More than one store per business

---

## License

© AgriTrade. All rights reserved.

This repository is a presentation of the project. The source code is private and is not licensed for reuse.
