# Customer Onboarding Flow
**Product:** Share — Hyper-local Clearance Deal Community  
**Version:** Phase 1 MVP  
**Date:** July 18, 2026  

---

## Overview

Share uses a **7-slide interactive onboarding flow** to introduce new users to the product immediately after sign-up. The onboarding is:

- **Auto-shown once** to every new user right after they create an account (via Google OAuth or email)
- **Skippable** at any time — users can jump straight to the feed
- **Re-accessible** at any time via the 📖 icon in My Deals (runs as a dismissable guide)
- **Never shown twice** — a `localStorage` flag marks it complete on first finish or skip

---

## Entry Points

| Trigger | Route | Behavior |
|---|---|---|
| New user signs up (Google or email) | `/onboarding` | Full 7-slide flow, Skip button visible |
| Returning user taps 📖 in My Deals | `/guide` | Same 7 slides, × close button instead of Skip |

---

## Onboarding Flow — Step by Step

### Slide 1 — Welcome

**Label:** 👋 Welcome  
**Headline:** Welcome to Share  
**Subheadline:** Spot deals. Share savings. Build trust.

**Body copy:**  
> Share is the hyper-local clearance deal community. Spot in-store deals with your phone, post them for your neighbors in real time, and build a reputation that matters.

**Visual:** Animated Share logo with a pulsing orange glow ring, all 6 reaction emojis animating in (🔥❤️👍😮💰🙌), and three key stats:
- **20mi** radius
- **Real** verified deals  
- **Free** forever

**Actions:**
- **Next →** — advances to Slide 2
- **Skip** — marks onboarding complete, goes to feed (new user flow only)

---

### Slide 2 — Scan Any Barcode

**Label:** 📱 Step 1  
**Headline:** Scan any barcode  
**Subheadline:** Auto-fill in seconds.

**Body copy:**  
> Point your camera at a product barcode and we pull the name, photo, and suggested retail price automatically. Add the clearance price — that's it. No manual typing needed.

**Visual:** Animated camera viewfinder with orange corner brackets and a scanning beam sweeping top to bottom, simulating a live barcode scan.

**Key detail for users:**  
Manual entry (typing product details without a barcode) is **locked by default**. It unlocks only after the user reaches a 5-star trust score — this keeps the feed high-quality.

**Actions:** Back · Next →

---

### Slide 3 — Upload a Receipt

**Label:** 🧾 Step 2  
**Headline:** Upload a receipt  
**Subheadline:** Earn the ✓ Verified badge.

**Body copy:**  
> Snap the clearance sticker or your receipt and upload it with your deal. This earns the Verified badge — proof your deal is real, not a screenshot from Discord.

**Visual:** A realistic receipt mockup (showing item name, clearance price, original price, discount %, store, date) with a green ✓ badge overlaid in the corner, and a "Verified badge earned" chip below.

**Why this matters:**  
The Verified badge is the primary trust signal in Share. It can only be earned with a receipt upload — there is no self-report toggle. This eliminates fake or inflated deal posts at the source.

**Actions:** Back · Next →

---

### Slide 4 — Trust Score

**Label:** ⭐ Trust Score  
**Headline:** Build your reputation  
**Subheadline:** 5 stars unlocks manual entry.

**Body copy:**  
> Every verified deal and every "it worked" confirmation from a buyer boosts your trust score. Hit 5 stars to unlock posting deals without a barcode scan.

**Visual:**
- 5-star row with animated fill
- Animated orange progress bar (showing example score 3.2 / 5.0)
- Formula card:

```
Verified ratio        × 3.5 pts
"It worked" count     × 1.5 pts
─────────────────────────────────
Max score               5.0 ★
```

**Formula in full:**
```
score = min(5.0,
  (verified_deals / total_deals) × 3.5
  + min(self_purchase_count, 20) / 20 × 1.5
)
```

**Actions:** Back · Next →

---

### Slide 5 — The Feed

**Label:** 🗺️ The Feed  
**Headline:** Deals within 20 miles  
**Subheadline:** Real. Local. Right now.

**Body copy:**  
> The feed shows active deals near you. React with 🔥, confirm "it worked" after you buy, or flag deals that are gone. Your activity helps the whole community.

**Visual:** Three animated deal cards showing real-looking listings (Dyson, KitchenAid, Sony) with price, discount %, reaction counts, and verified badges — cascading in with a stagger animation.

**Actions:** Back · Next →

---

### Slide 6 — How Deals Stay Fresh

**Label:** 📋 Good to know  
**Headline:** How deals stay fresh  
**Subheadline:** Auto-expiry keeps the feed honest.

**Visual:** Five rule cards, each animating in with a stagger:

| Icon | Rule | Detail |
|---|---|---|
| ⚠️ | **Auto-expiry** | 4h grocery · 24h big-box. Deals vanish automatically. |
| ✅ | **3-flag remove** | 3 community "expired" flags removes the deal instantly. |
| 🔔 | **"Still Here?" ping** | You'll get a prompt 2h before your deal expires. Tap to extend. |
| 🛒 | **Buy-for-me signal** | Tapping "Want pickup" is a passive intent signal — no transaction yet. |
| 🔥 | **Trending** | 5+ "it worked" confirmations auto-promotes your deal to trending. |

**Actions:** Back · Let's go! →

---

### Slide 7 — You're All Set

**Label:** 🚀 Ready  
**Headline:** You're all set!  
**Subheadline:** Start spotting deals for your neighbors.

**Body copy:**  
> The feed is live and waiting. Browse what your neighbors have spotted — or head straight to the scanner to post your first deal.

**Visual:** None — clean CTA focus.

**Actions (two primary buttons):**
1. **📷 Scan My First Deal** — goes to `/scan` (protected, requires sign-in — already complete at this point)
2. **Browse the Feed First** — goes to `/` (public feed)

Both actions mark onboarding as seen in `localStorage`.

---

## Navigation Rules

| Situation | Behavior |
|---|---|
| First slide, new user flow | Skip button (top right) → marks seen, goes to feed |
| First slide, guide flow (`/guide`) | × close button → marks seen, goes back |
| Slides 2–6 | Back button (top right) + Next button (bottom) |
| Last slide | Back button + two CTA buttons (no Next) |
| Dot indicators | Top left — 7 dots, active dot grows wider (animated) |
| Slide transitions | Horizontal slide animation (60px), 280ms ease-in-out |

---

## State Management

```
localStorage key: 'share-onboarding-seen'
Value when seen:  '1'
Set on:           Completing slide 7, tapping Skip, or tapping × in guide mode
Checked by:       App.tsx (afterSignUpUrl redirect only — no forced redirect for existing users)
```

**New user redirect:**
- Clerk `afterSignUpUrl` is set to `/onboarding`
- After email verification completes, Clerk sends the user directly to the onboarding flow
- Sign-in (existing users) goes to the feed as normal — onboarding is not forced again

---

## Routes

| Route | Component | Props | Access |
|---|---|---|---|
| `/onboarding` | `Onboarding` | `dismissable={false}` | Public (no auth required) |
| `/guide` | `Onboarding` | `dismissable={true}` | Public (no auth required) |

Both routes are registered in `App.tsx` alongside the Wouter `<Switch>`.

---

## Design Specifications

| Property | Value |
|---|---|
| Background | `#020617` (slate-950) |
| Primary accent | `#f97316` (orange-500) |
| Secondary accent | `#10b981` (emerald-500) — receipt / verified slide |
| Tertiary accent | `#eab308` (yellow-500) — trust score slide |
| Font | Outfit (variable weight 400–900) |
| Slide animation | Framer Motion `AnimatePresence`, x ±60px, 280ms ease-in-out |
| Progress dots | Orange pills, active dot animates width 6px → 20px |
| Primary button | Orange-500, rounded-2xl, full width, `font-black`, shadow-xl |
| Secondary button | Slate-900, border slate-800, rounded-2xl (last slide only) |
| Card radius | `rounded-2xl` (16px) throughout |
| Max width | 448px (max-w-md), centered, full-height |

---

## Accessibility & UX Notes

- All slides support **Back** navigation — users are never trapped
- **Skip** is prominent on slide 1 so users who already understand the app aren't forced through it
- The receipt/verification explanation (Slide 3) is critical — early user research on similar apps shows confusion about why a receipt is needed vs. a product photo. The slide copy addresses this directly ("proof your deal is real, not a screenshot from Discord")
- The trust score formula (Slide 4) is shown visually, not just described — reduces "why can't I post manually yet?" support questions
- The rules slide (Slide 6) front-loads the "Still Here?" ping mechanic — the most common surprise for new posters is getting an expiry confirmation prompt

---

## Re-entry: "How it Works" in My Deals

Users can re-read the onboarding at any time via the **📖 (book) icon** in the top-right corner of the My Deals page. This opens `/guide`, which renders the same 7 slides with a **×** dismiss button (no Skip label) that returns the user to the previous page.

This doubles as the app's **in-product user documentation** — no separate help center or external docs page needed for Phase 1.

---

*Customer Onboarding Flow · Share Phase 1 MVP · July 18, 2026*
