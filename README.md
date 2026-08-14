# Level Up

A production iOS app that turns a day of focus, training, and execution into a scored judgment — designed, engineered, and shipped independently.

<p align="center">
  <img src="assets/app-icon.jpg" alt="Level Up app icon" width="96" height="96">
</p>

**Live product:** [App Store](https://apps.apple.com/app/levelup-ai/id6756843363) · [levelupself.app](https://levelupself.app)

This repository is a **public engineering case study**. The production source stays private. I can walk hiring managers through the real codebase on request.

---

## Product

Level Up is a self-improvement app for people who want structure instead of motivation. Users plan the day, run focus sessions, log training, and receive an AI Judge verdict grounded in what they actually did.

The App Store listing is live under seller **Wassim Akkash** (`LevelUp - AI`, version 1.0, released August 2026).

Core product surfaces:

- Daily planning and execution
- Focus timers
- Workout tracking and an interactive muscle map
- Four pillars: Focus, Strength, Discipline, Energy
- AI Judge for daily accountability
- Premium subscription for expanded AI usage

---

## Engineering

| Area | What shipped |
| --- | --- |
| Client | Flutter / Dart, iOS-first, Riverpod, GoRouter |
| Local data | Drift / SQLite, outbox-based sync |
| Backend | Supabase (Auth, PostgreSQL, Realtime, Edge Functions) |
| Identity | Email, Sign in with Apple, Google Sign-In |
| Billing | StoreKit via RevenueCat — offerings, entitlements, restore |
| Health | Apple Health (HealthKit) for optional sleep → Biological Energy |
| Scoring | Deterministic engines for user-visible numbers; AI writes verdict prose, not scores |
| Observability | Sentry, structured error handling |
| Web / compliance | Custom domain, privacy, terms, support, account deletion |

AI calls go through services and Edge Functions with typed request/response contracts. The UI does not call the model directly.

---

## Architecture

Feature-first Flutter client, with a service layer between UI and backend.

```text
┌─────────────────────────────────────────────────────────┐
│                     iOS app (Flutter)                    │
│  UI  →  Riverpod  →  Services  →  Local DB (Drift)      │
│                         │                                │
│                    Outbox / sync                         │
└─────────────┬───────────────┬───────────────┬────────────┘
              │               │               │
              ▼               ▼               ▼
        Supabase         RevenueCat      Apple Health
     Auth · Postgres      StoreKit         HealthKit
     Realtime · Edge
              │
              ▼
     levelupself.app (legal, support, deletion)
```

Local writes are durable first. Remote sync is asynchronous. User-visible scores have a declared canonical owner so Stats, Judge, and Home cannot invent competing numbers.

The production app follows a strict layering: **UI → providers → services → data**. Database access, billing, and AI inference are not done from widgets.

---

## Engineering Challenges

### 1. Deterministic scores vs AI language

**Problem.** An AI Judge that both narrates the day and invents the score will drift. Two screens would disagree; App Review and users would not trust the number.

**Decision.** Split authority. Math owns Focus, Strength, Discipline, Energy, and related display scores. A formula registry records the canonical owner, inputs, range, and rounding. AI may write verdict prose and a tone band. It must not own the number the user sees.

**Result.** Scoring is testable. Judge Chat shows tone, not a competing `/10`. Stats and judgment stay aligned.

### 2. Local-first work vs remote truth

**Problem.** Focus sessions and workouts cannot disappear because the network dropped. Multi-device and retry also cannot double-apply the same session.

**Decision.** Persist locally (SQLite), enqueue mutations in an outbox, sync when the network is available, and reconcile with remote rows. Realtime is used to refresh, not as the source of truth.

**Result.** Execution still records offline. Sync is an engineering problem with tests, not a spinner around a live query.

### 3. Subscriptions that match the App Store

**Problem.** StoreKit, restore, trial, expiry, and premium gates fail in ways that look like product bugs. A second billing path would split truth.

**Decision.** Ship iOS billing through RevenueCat and a single premium entitlement. Cache entitlement state for launch, restore through the store, and keep paywall copy store-safe.

**Result.** Premium is an entitlement, not a flag flipped in the client.

### 4. HealthKit without leaking health data into AI

**Problem.** Apple Review rejects vague HealthKit use. Sending raw sleep samples to a model is both a privacy and a review failure.

**Decision.** HealthKit is optional. Permitted sleep reads feed Biological Energy. Raw samples, stages, timestamps, and HealthKit identifiers are not sent to the model. Users can continue without connecting Apple Health. Privacy copy, system usage strings, and contract tests have to match.

**Result.** The HealthKit path is reviewable: optional, disclosed, and bounded.

### 5. Shipping through App Review, not past it

**Problem.** A Flutter app with AI, HealthKit, subscriptions, and account deletion will be rejected on process, not only on crashes.

**Decision.** Treat review as an engineering surface: privacy nutrition labels, in-app and web account deletion, AI disclosure, subscription terms, signed release builds, and written review responses for successive builds.

**Result.** Version 1.0 is on the App Store. The legal site is live for the same product.

---

## Testing & Reliability

The production repository (private) contains:

- Hundreds of Dart unit/widget/contract tests
- Backend/Edge Function tests
- GitHub Actions for analyze, test, release, and secrets presence
- Contract tests for scoring ownership, HealthKit privacy strings, and billing identifiers
- Sentry for production failures

This is not a screenshot-only prototype. Scoring, entitlements, HealthKit disclosures, and sync behavior are asserted in tests.

---

## Shipping to Production

Level Up is a released product, not a campus demo.

| Gate | What exists |
| --- | --- |
| Store | App Store listing, seller Wassim Akkash, version 1.0 |
| Billing | App Store Connect products + RevenueCat offering/entitlement |
| Privacy | Privacy policy, terms, AI disclosure, HealthKit usage strings |
| Deletion | In-app flow and [levelupself.app/delete-account](https://levelupself.app/delete-account) |
| Release | Signed iOS archives, build numbers, App Review remediation |
| Ops | Edge Functions, Sentry, CI |

---

## Web Platform

**[levelupself.app](https://levelupself.app)** is the compliance and support site, not a second app.

Verified live:

- Custom domain and HTTPS
- Privacy, terms, support
- Account deletion instructions
- Deployed as a static site (Vercel)

It exists because App Store distribution requires reachable legal and deletion URLs. Design, copy, hosting, DNS, and SSL were part of shipping the product.

---

## Tech Stack

**Client:** Flutter, Dart, Riverpod, GoRouter, SQLite  
**iOS:** StoreKit, Sign in with Apple, HealthKit  
**Backend:** Supabase, PostgreSQL, Row-Level Security, Edge Functions  
**Billing:** RevenueCat  
**Quality:** automated tests, GitHub Actions, Sentry

---

## What I Owned

I built Level Up independently: product design, Flutter/iOS engineering, backend, billing, HealthKit, testing, App Store submission, and the legal website.

The App Store seller name and the git author are the same person. There was no separate mobile team, backend team, or design agency.

---

## Why the source is private

Level Up is a live commercial product. Publishing the full source would expose proprietary scoring, AI contracts, billing catalogs, and backend attack surface. A case study plus a live App Store build is the honest portfolio. I will walk through architecture and code in interviews.

---

## Links

- App Store: [LevelUp - AI](https://apps.apple.com/app/levelup-ai/id6756843363)
- Website: [levelupself.app](https://levelupself.app)
- LinkedIn: _add your profile URL_

This repository does not accept public contributions. It is a portfolio case study, not an open-source application.
