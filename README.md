# Level Up

An iOS app for planning, training, focus, and AI-assisted review — designed, engineered, and shipped independently.

<p align="center">
  <img src="assets/app-icon.jpg" alt="Level Up app icon" width="96" height="96">
</p>

**Live product:** [App Store](https://apps.apple.com/app/levelup-ai/id6756843363) · [levelupself.app](https://levelupself.app)

This repository is a **public engineering case study**. The production source stays private.

## Runnable engineering examples

For code, tests, and reproducible failure scenarios, start here:

- [LLM structured-output pipeline](https://github.com/wassim991/llm-structured-output-pipeline): schema validation, bounded repair, typed failures, and cancellation in TypeScript/Deno.
- [Flutter offline sync demo](https://github.com/wassim991/flutter-offline-sync-demo): durable SQLite writes, an outbox, idempotent retry, realtime reconciliation, and an SSE lab.

These are standalone demonstrations informed by LevelUp engineering work, not copies of the production implementation. Their READMEs distinguish real local behavior from simulated external services.

---

## Product

Level Up is a self-improvement app for people who want structure instead of motivation. Users plan the day, run focus sessions, log training, and receive an AI Judge verdict grounded in what they actually did.

The App Store listing is under seller **Wassim Akkash** (`LevelUp - AI`). See the listing for current version and availability.

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

The public examples linked above provide runnable tests. This section describes the private app; its full test suite is not included in this repository.

---

## Shipping to Production

Release work covered the following surfaces:

| Gate | What exists |
| --- | --- |
| Store | App Store listing, seller Wassim Akkash |
| Billing | App Store Connect products + RevenueCat offering/entitlement |
| Privacy | Privacy policy, terms, AI disclosure, HealthKit usage strings |
| Deletion | In-app flow and [levelupself.app/delete-account](https://levelupself.app/delete-account) |
| Release | Signed iOS archives, build numbers, App Review remediation |
| Ops | Edge Functions, Sentry, CI |

---

## Web Platform

**[levelupself.app](https://levelupself.app)** presents the product and hosts its privacy, terms, support, and account-deletion information.

Website responsibilities:

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

---

## Source availability

LevelUp is a commercial product and its production source remains private. The linked public repositories demonstrate selected engineering patterns with generalized data, documented trade-offs, and executable tests.

---

## Links

- App Store: [LevelUp - AI](https://apps.apple.com/app/levelup-ai/id6756843363)
- Website: [levelupself.app](https://levelupself.app)

This repository documents the product. For implementation details and runnable tests, use the engineering examples linked above.
