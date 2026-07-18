# Share — Design Document & Tech Stack
**Date:** July 18, 2026  
**Version:** Phase 1 MVP

---

## 📄 Design Document

### Vision
A community-powered, hyper-local deal discovery platform where real shoppers verify and share in-store clearance deals in real time, with a built-in "buy for me" relay system (Phase 2).

---

### Goals

| Goal | Detail |
|---|---|
| Trusted feed | Receipt-verified deals that eliminate stale/fake posts |
| Social currency | Views, reactions, reputation for deal hunters |
| Hyper-local | 15–20 mile radius, actionable deals |
| Phase 2 groundwork | C2C pickup relay activated only once demand and trust are proven |
| Monetization | Free for users forever — revenue from businesses, not consumers |

---

### User Types

**User A — The Deal Hunter** (25–45, budget-conscious, already in-store)
- Motivated by social recognition, reputation, gamified progression
- Pain: shares deals on fragmented platforms (Discord, FB groups) with no verification

**User B — The Deal Seeker** (25–50, time-poor, price-sensitive)
- Motivated by access to real, current deals without going in-store
- Pain: stale posts, drives to store and item is gone

---

### Core Features (Phase 1 MVP — Built)

| Feature | Description |
|---|---|
| Barcode Scanner | Scan → auto-fill product name, image, price via Open Food Facts |
| Manual Entry (gated) | Unlocked only at 5-star trust score — enforced in UI and API |
| Deal Listing | Store, clearance price, aisle, photo, expiry, category |
| Receipt Upload | Separate from product photo — required for the ✓ verified badge |
| Deal Feed | Browse within 20 miles, filter by store/category, geo-sorted |
| Poster Confirmation Pings | Deals flagged `needsConfirmation` when within 2h of expiry — poster prompted in MyDeals |
| Social Layer | View counts, 6-emoji reactions (🔥❤️👍😮💰🙌), "it worked" confirmations, buy-for-me interest |
| Trust Score | Live formula: `(verified/total)×3.5 + min(selfPurchaseCount,20)/20×1.5`, capped at 5.0 |
| Auto-expiry | 4h grocery / 24h big-box — community flags accelerate expiry at 3+ |
| Auth & Identity | Clerk (Google OAuth + email verification), persistent userId on all deals |

### Phase 2 / 3 (Deferred — not built)
- Buy-for-me relay (activated by demand signal count)
- Push notifications, route planner, ML expiry predictions
- Cashback/tips for User A, B2C data licensing

---

### Deal Lifecycle

```
User A in-store
  → scans barcode (or manual entry if trust ≥ 5★)
  → uploads receipt → verified badge earned
  → posts deal → live in feed for 15-20 mi radius

User B sees deal:
  👀 view  → increments User A's view count
  🔥 react → builds reputation
  ✅ "it worked" → resets expiry + builds trending score
  🛒 buy-for-me interest → passive Phase 2 demand signal
  🚩 flag expired/gone → 3 flags = auto-expire

Poster gets pinged 2h before expiry → "Still Here?" button
```

---

### Trust Score Formula

```
score = min(5.0,  (verified_deals / total_deals) × 3.5
                + min(self_purchase_count, 20) / 20 × 1.5)
```

- **3.5 pts** from verified ratio (receipt-backed)
- **1.5 pts** from buyer confirmations (capped at 20)
- **5.0** unlocks manual entry

---

### Monetization (Staged)

| Phase | Model |
|---|---|
| 1 | Zero revenue — prove the feed works |
| 2 | Take rate on completed Buy-for-Me transactions |
| 3 | B2C aggregated trend data licensing to retailers |

> No ads. Ever.

---

### Competitor Landscape

| Competitor | Strength | Weakness vs. Share |
|---|---|---|
| PennyForge | Trust engine, route planning | No buy-for-me relay, no C2C layer |
| BarcodeVibe | Clean UX, price history | No community/social layer |
| Cartive | Business tools, 10-mile radius | Business-led, no user verification |
| Flipp | Massive store partnerships | No real-time in-store verification |
| Reddit/Discord | Engaged communities | Unstructured, no verification, deals go stale |

**Share's unique angle:** C2C relay + receipt-verified + barcode-scanned + social reputation + free, ad-free, hyper-local.

---

### Risk Analysis

#### High Risk (Phase 1 Mitigations)

| Risk | Mitigation |
|---|---|
| Cold start | Cut Buy-for-Me from Phase 1. Seed from Reddit/Discord deal communities. One user type = no chicken-and-egg problem. |
| Deal staleness | "Already in-store" positioning. 4h/24h auto-expiry. `needsConfirmation` pings. 3-flag community expiry. |
| Scam in buy-for-me | Eliminated entirely — no money changes hands in Phase 1. |
| Store pushback | User-generated only. No scraping. No store logos. Non-issue at MVP scale. |

#### Medium Risk

| Risk | Mitigation |
|---|---|
| Low User A motivation | Social currency (views, reactions, trust score progression). Badges/leaderboards planned. |
| Geographic fragmentation | Launch single metro, build density before expanding. |
| Barcode data accuracy | Open Food Facts primary. Manual correction allowed. Second DB (UPC Database) planned. |
| Location denial | ZIP/city fallback UX planned (M4 gap). |

---

### Phase 1 Success Metrics

1. Deal posting rate per active User A (posts/week)
2. Feed engagement: view → reaction ("thank you") conversion rate
3. "It worked" confirmation rate from feed browsers
4. User A repeat-posting / retention rate
5. Time from deal posted → first reaction

**Phase 1 → Phase 2 Trigger:** Volume of users tapping "Request Buy-for-Me" interest button (collected but relay not activated until threshold is met).

---

### Gap Analysis — Strategy Doc vs. Build

| Severity | ID | Gap | Status |
|---|---|---|---|
| 🔴 Critical | C1 | No user authentication or persistent identity | ✅ Fixed — Clerk (Google + email) |
| 🔴 Critical | C2 | Manual entry ungated — available to all users | ✅ Fixed — trust score ≥ 5.0 gate |
| 🔴 Critical | C3 | Receipt upload missing; verified = self-reported boolean | ✅ Fixed — `receiptUrl` required to earn badge |
| 🟠 High | H1 | View count not incrementing on deal view | ✅ Fixed — `GET /deals/:id` increments DB |
| 🟠 High | H2 | No poster confirmation pings for near-expiry deals | ✅ Fixed — `needsConfirmation` flag + MyDeals prompt |
| 🟠 High | H3 | Geo radius hard-coded; no UI indicator | 🔲 Partial — named constant, UI label present |
| 🟠 High | H4 | "It worked" count absent from feed cards | ✅ Fixed — `selfPurchaseCount` on cards; trending at ≥5 |
| 🟡 Medium | M1 | No badges or leaderboard | 🔲 Deferred (needs auth first — now unblocked) |
| 🟡 Medium | M2 | Only one barcode data source (Open Food Facts) | 🔲 Deferred — second DB fallback planned |
| 🟡 Medium | M3 | No Phase 1 metrics instrumentation / admin endpoint | 🔲 Deferred |
| 🟡 Medium | M4 | Location denial has no UX fallback | 🔲 Deferred — ZIP entry fallback planned |

#### ✅ Built Beyond Strategy Spec

| Feature | Notes |
|---|---|
| 6-emoji reaction set (🔥❤️👍😮💰🙌) | Strategy mentions "emoji reactions" generically |
| 3-flag auto-expire | Sensible default not specified in strategy doc |
| storeType-based expiry windows | 4h grocery / 24h big-box — elaboration of "4–24h configurable" |
| Magic-byte photo validation | Security hardening beyond what the strategy specified |
| PickupRequests table / Phase 2 endpoints | Infrastructure-only, forward-compatible |

---

## 🛠 Tech Stack

### Monorepo

| Tool | Purpose |
|---|---|
| **pnpm workspaces** | Monorepo package manager |
| **TypeScript** | Across all packages |

---

### Frontend — `artifacts/share-app`

| Library | Version | Purpose |
|---|---|---|
| **React** | 19 (catalog) | UI framework |
| **Vite** | 7 (catalog) | Dev server + bundler |
| **Tailwind CSS** | 4 (catalog) | Utility-first styling |
| **@clerk/react** | ^6.12.5 | Auth — Google OAuth + email verification |
| **@clerk/themes** | ^2.4.57 | `shadcn` dark theme base for Clerk UI |
| **Wouter** | ^3.3.5 | Lightweight client-side routing |
| **TanStack Query** | catalog | Server state, caching, mutations |
| **Framer Motion** | catalog | Animations |
| **Radix UI** (full suite) | various | Headless accessible component primitives |
| **Lucide React** | catalog | Icon set |
| **shadcn/ui** | custom | Component system built on Radix + Tailwind |
| **Web BarcodeDetector API** | browser-native | Camera barcode scanning (no library needed) |
| **react-hook-form** | ^7.55.0 | Form state management |
| **zod** | catalog | Schema validation |

---

### Backend — `artifacts/api-server`

| Library | Version | Purpose |
|---|---|---|
| **Express** | ^5.2.1 | HTTP server |
| **@clerk/express** | ^2.1.43 | Session verification middleware (`getAuth`) |
| **@clerk/shared** | ^4.25.5 | `publishableKeyFromHost` resolution |
| **http-proxy-middleware** | ^4.2.0 | Clerk FAPI proxy (required for Replit-hosted Clerk) |
| **Drizzle ORM** | catalog | Type-safe SQL query builder |
| **Pino** | ^9.14.0 | Structured JSON logging |
| **pino-http** | ^10.5.0 | Request-level logging middleware |
| **cors** | ^2.8.6 | Cross-origin headers with `credentials: true` |
| **esbuild** | 0.27.3 | Fast TypeScript bundler for production dist |
| **Open Food Facts API** | external (free) | Barcode → product name / image lookup |

---

### Database — `lib/db`

| Tool | Purpose |
|---|---|
| **PostgreSQL** | Replit-managed hosted database |
| **Drizzle ORM** | Schema definition and type-safe query builder |
| **drizzle-kit** | Push-based schema migrations (`drizzle-kit push --force`) |
| **drizzle-zod** | Auto-generates Zod validators from Drizzle schema |
| **pg** | ^8.22.0 — Node.js PostgreSQL client |

#### Tables

| Table | Key Columns |
|---|---|
| `deals` | `id`, `userId` (Clerk), `itemName`, `originalPrice`, `clearancePrice`, `discountPercent`, `storeName`, `storeAddress`, `aisleLocation`, `storeType`, `category`, `photoUrl`, `receiptUrl`, `verified`, `viewCount`, `requestCount`, `selfPurchaseCount`, `buyForMeInterestCount`, `flagCount`, `status`, `trending`, `needsConfirmation`, `posterName`, `postedAt`, `expiresAt`, `lastGrabbedAt`, `latitude`, `longitude` |
| `pickup_requests` | `id`, `dealId`, `pickupTime`, `message`, `requesterName`, `status`, `createdAt` |
| `deal_reactions` | `id`, `dealId`, `emoji`, `createdAt` |
| `deal_flags` | `id`, `dealId`, `reason` (`expired` \| `inventory`), `createdAt` |

---

### Shared Libraries

| Package | Purpose |
|---|---|
| `@workspace/api-zod` | Shared Zod schemas for all API request/response bodies |
| `@workspace/api-client-react` | Auto-generated TanStack Query hooks; custom fetch with `credentials: include` |
| `@workspace/db` | Drizzle schema + DB client — exported to all packages |

---

### Infrastructure

| Service | Detail |
|---|---|
| **Replit** | Hosting, managed PostgreSQL, secrets, preview proxy, pnpm monorepo |
| **Clerk** | Replit-managed tenant — Google OAuth + email/password + email verification. No external Clerk dashboard needed. |
| **File uploads** | `POST /api/upload` — stored via Replit Object Storage; magic-byte MIME validation; 5 MB limit |

---

### API Surface (Key Routes)

| Method | Path | Auth | Purpose |
|---|---|---|---|
| `GET` | `/api/deals` | Public (optional) | List active deals; `?mine=true` returns poster's own deals (all statuses) |
| `POST` | `/api/deals` | Required | Create deal — `userId` from Clerk session; `receiptUrl` sets `verified: true` |
| `GET` | `/api/deals/stats` | Required | Trust score + stats scoped to authenticated user |
| `GET` | `/api/deals/:id` | Public | Get deal + increment `viewCount` |
| `DELETE` | `/api/deals/:id` | Required (owner) | Delete own deal |
| `PATCH` | `/api/deals/:id/verify` | Required (owner) | Add `receiptUrl` → sets `verified: true` |
| `PATCH` | `/api/deals/:id/confirm-still-here` | Required (owner) | Reset `needsConfirmation`, extend expiry window |
| `POST` | `/api/deals/:id/self-purchase` | Public | Record "it worked"; auto-promotes `trending: true` at 5+ |
| `POST` | `/api/deals/:id/react` | Public | Add emoji reaction (🔥❤️👍😮💰🙌) |
| `POST` | `/api/deals/:id/flag` | Public | Flag expired/gone; auto-expire at 3+ flags |
| `POST` | `/api/deals/:id/buy-for-me-interest` | Public | Passive Phase 2 demand signal |
| `GET` | `/api/barcode/:barcode` | Public | Open Food Facts barcode lookup proxy |
| `POST` | `/api/upload` | Public | Photo/receipt upload → Object Storage URL |
| `POST` | `/api/deals/:id/requests` | Public | Request pickup (Phase 2 infrastructure) |

---

### Auth Flow

```
Browser (Clerk) → Google OAuth / Email+Password
  → Clerk issues session cookie (HttpOnly)
  → All fetch calls include credentials: 'include'
  → API: clerkMiddleware() verifies session on every request
  → requireAuth middleware extracts userId via getAuth(req)
  → userId stored on deals; all trust/stats scoped to userId
```

Protected routes: `/scan`, `/my-deals`  
Public routes: `/`, `/deals/:id`, `/sign-in`, `/sign-up`

---

*Document generated July 18, 2026 · Share Phase 1 MVP*
