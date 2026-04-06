# ForTicket — Full-Stack Event Ticketing Platform

[![PHP](https://img.shields.io/badge/PHP-8.2-777BB4?logo=php&logoColor=white)](https://php.net)
[![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)](https://mysql.com)
[![TailwindCSS](https://img.shields.io/badge/TailwindCSS-3.4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![License](https://img.shields.io/badge/License-Private%20%2F%20Commercial-red)](.)

---

## Overview

**ForTicket** is a production-grade, multi-tenant event ticketing platform built from scratch for a commercial client. It enables event organizers to create and sell tickets online and at physical points of sale, manage attendees, generate financial reports, and scale their operations — all from a single, branded platform.

The system handles the complete lifecycle of an event: from **discovery and ticket purchase** by end users, to **check-in, financial settlement, promoter commissions, and photo coverage** — replacing fragmented tools with a unified, self-hosted solution.

> Estimated to serve hundreds of concurrent users, processing both PIX and international card payments with real-time order confirmation and automated ticket delivery.

---

## Code Availability

> **This project is private due to commercial client confidentiality.**
> Source code is not publicly available. This repository exists as a portfolio reference only.
> A live demo or technical walkthrough is available upon request.

---

## Tech Stack

| Layer | Technologies |
|---|---|
| **Backend** | PHP 8.2, PDO, PSR-4 Autoloading via Composer |
| **Frontend** | Tailwind CSS 3.4, Alpine.js 3, Lucide Icons, Plus Jakarta Sans |
| **Database** | MySQL 8 (primary), with optional SQLite for local development |
| **Payments** | MercadoPago SDK (PIX, Boleto, Credit Card), Stripe, PayPal |
| **Email** | PHPMailer 6 over SMTP (configurable per environment) |
| **Auth** | Session-based authentication + Google OAuth 2.0 |
| **Security** | CSRF tokens, HttpOnly/SameSite session cookies, parameterized queries |
| **Tooling** | Composer, npm / Tailwind CLI, `.env`-based configuration |

---

## Key Features

### For Event Attendees
- **Instant discovery** — browse events by city, region, or category with a hero slider, city cards, and a favorites system
- **Geolocation** — automatic nearest-city detection via IP/browser API
- **Secure checkout** — supports PIX, bank slip (boleto), and credit/debit card through MercadoPago; Stripe and PayPal also configurable
- **Promoter coupon codes** — discount codes with server-side validation and abuse prevention
- **Digital tickets** — downloadable PDF tickets with unique cryptographic codes (generated with `random_bytes`) and all buyer/event data
- **Google SSO** — one-click sign-in with state-token CSRF protection on the OAuth flow
- **Account self-service** — password reset via email, profile management, GDPR-style account deletion with a 7-day grace period

### For Event Organizers (Company Portal)
- **Multi-tenant dashboard** — each company has an isolated panel with branded layouts
- **Event management** — create, edit, duplicate, and delete events; manage ticket types in batches with price tiers and capacity limits
- **Custom registration forms** — drag-and-drop field builder for collecting attendee data (CPF, gender, address, multiple-choice, etc.)
- **Point of Sale (PDV)** — full in-person sales terminal for box-office and gate operations, including product/merchandise sales
- **QR-code check-in** — real-time attendee validation with collaborator access control
- **Financial reports** — per-event revenue, pending payments, promoter commissions, and exportable reports
- **Promoter / affiliate system** — approve promoters, assign them to events, generate trackable referral links, and calculate commissions automatically
- **Coupon management** — create percentage or fixed-value discount coupons scoped per event
- **CRM / Interest lists** — capture and export leads before ticket sales open
- **Team collaboration** — invite collaborators by email with granular event-level permissions
- **Photographer portal** — dedicated module for event photographers to upload and organize coverage photos per event
- **Third-party integrations** — per-event webhook and integration settings

### For Platform Administrators (Super Admin Panel)
- **Full user and company management** — activate/deactivate accounts, switch organizer context
- **Event moderation** — approval workflow before events go public
- **Global settings** — configure SMTP, payment gateway keys, Google OAuth credentials, banners, and platform-wide assets — all through a secure UI without touching code
- **City and region management** — geographic structure powering discovery and geolocation
- **Lead and promoter oversight** — platform-wide visibility across all organizations

---

## Architecture & Technical Highlights

```
public_html/
├── admin/               # Super-admin panel (role-gated)
├── api/                 # JSON endpoints (payments, cart, coupons, webhooks)
├── company/             # Organizer portal (~40 modules)
├── photographer/        # Dedicated photographer portal
├── config/              # DB connection, environment loading
├── includes/            # Shared helpers, mailer, CSRF, header/footer
├── assets/              # Compiled Tailwind CSS, images
├── database/            # Schema migrations (run on deploy)
└── vendor/              # Composer dependencies
```

**Multi-tenancy via company isolation** — every organizer's data is scoped by `company_id` at the database level with join-layer authorization checks on every query. No shared sessions between tenants.

**Payment abstraction** — the checkout module detects which payment provider is configured and renders the appropriate UI (MercadoPago Bricks, Stripe Elements, or PayPal). Webhooks from each provider (`mp_webhook.php`, `stripe_webhook.php`, `paypal_ipn.php`) handle asynchronous payment confirmation independently, updating order status without blocking the user flow.

**Schema migrations** — database changes are managed through `database/migrate.php`, keeping production deploys predictable. Runtime column checks (`db_ensure_column`) provide a safe fallback for incremental schema updates.

**Security-first design** — all user input goes through PDO prepared statements. Session cookies are configured with `HttpOnly`, `SameSite=Lax`, and `Secure` flags auto-detected from the server environment (including Cloudflare proxy headers). File uploads are validated by MIME type via `finfo`, not file extension.

**Affiliate tracking** — referral links inject a `ref` token at any page entry point. Clicks are logged and attributed to the correct promoter before any purchase is committed, using a non-blocking try/catch that never degrades the checkout experience.

---

## Challenges & Solutions

| Challenge | Solution |
|---|---|
| Supporting 3 distinct payment providers with different APIs and async confirmation flows | Modular payment files (`mp_process_payment.php`, `stripe_process_payment.php`, `paypal_*.php`) with a unified order schema and individual webhooks per provider |
| Preventing race conditions in ticket inventory | Database-level quantity tracking with transactional order creation; ticket codes generated with `random_bytes` to prevent guessing |
| Multi-tenant data isolation without a framework | All queries scope by `company_id` with verified JOINs; a shared `auth_company.php` middleware guards every organizer route |
| Gradual schema evolution in production without downtime | `database/migrate.php` for planned migrations + `db_ensure_column()` for safe incremental column additions at runtime |
| Keeping checkout fast under varying network conditions | Server-side cart total calculation prevents client-side tampering; MP Bricks and Stripe Elements load asynchronously after page paint |

---

## Business Impact

- **Replaces 3–5 external tools** (ticketing service, email platform, PDV app, affiliate tracker, photo gallery) with a single owned platform — eliminating per-ticket SaaS fees
- **PIX integration** reduces payment friction for the Brazilian market where PIX has >70% adoption in digital payments, directly increasing conversion rates
- **Affiliate system** turns promoters into a distribution channel at zero upfront cost — commissions are only paid on confirmed sales
- **PDV module** enables organizers to sell tickets at physical venues, capturing revenue that would otherwise be lost to cash queues or no-shows
- **Real-time analytics** give organizers instant visibility into sales velocity, allowing dynamic pricing decisions before events sell out
- **Automated ticket delivery** via email with QR code reduces support volume by eliminating "where is my ticket?" inquiries

---

## Screenshots

> Screenshots available upon request or during a live demo session.

---

## What I Built & Learned

- Architected and delivered an end-to-end commercial SaaS product as the sole backend engineer, from database schema to production deployment
- Integrated three payment gateways (MercadoPago, Stripe, PayPal) on a single checkout flow with webhook-driven order confirmation — handling PIX's asynchronous nature without degrading UX
- Designed a multi-tenant authorization system without an ORM or framework, relying on disciplined query scoping and middleware guards
- Built a promoter affiliate system with click attribution, commission calculation, and an approval workflow from scratch
- Implemented a configurable, database-driven settings system that allows non-technical admins to manage SMTP credentials, payment keys, and OAuth secrets through a UI — removing deployment dependencies for configuration changes
- Gained deep experience with PDO security patterns, schema migration strategies, and session security hardening in PHP 8.2

---

## Contact

Interested in this project or want to discuss similar work?

**Marco Nunes** — [LinkedIn](https://linkedin.com/in/marconunes) · [Email](mailto:contato@marconunes.dev)

> Demo access available upon request.
