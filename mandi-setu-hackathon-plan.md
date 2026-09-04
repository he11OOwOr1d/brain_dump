# MANDI-SETU — 1-Week Internal Hackathon Plan (SIH 2026 · PS 26132)

> **Inputs reconciled:** the 54-page playbook (MANDI-SETU), three prior sprint plans, and the actual constraint set: **1 week, 6 people, no pre-work, college judges, real-but-flaky data.**

---

## 0. The one decision that drives everything

**Scope: Maharashtra only. Wedge: Onion in the Nashik–Ahmednagar–Pune belt, Lasalgaon as anchor.**

The pan-India question is settled. Build for Maharashtra because the problem statement is *issued by* the Government of Maharashtra, one regulator (MSAMB / ~295 APMCs), one language (Marathi), and one dialect family. A pan-India build is a shallower answer to a specific question. And inside Maharashtra, do **not** build "all crops" — build **onion deep**, and show the schema generalises.

**Why onion wins over the cotton idea in your earlier sprint doc:** the playbook's core thesis is that the binding constraint is the *ability to wait*, not information. Onion has no MSP, extreme volatility, is storable (so HOLD is physically real), and Lasalgaon is Asia's largest onion market. Cotton has an MSP floor — which *caps* the value of a sale-window engine. Cotton is expansion #2, not the wedge.

**The one-sentence thesis (memorise this):** *everyone else builds a price dashboard; we build a "waiting machine" — forecast + storage + pledge credit as one loop, with information as the input that makes the waiting decision correct.*

---

## 1. Blind spots (the thing you asked me to find)

These are split into the playbook's blind spots (still critical, kept short) and the ones **the playbook does NOT cover for a 1-week, no-pre-work, college-internal hackathon** — those are the ones most likely to sink you.

### 1a. The playbook's blind spots (don't forget these)

1. **Dashboard ≠ money.** A dashboard shows a price; it doesn't put rupees in a hand. The obsession is *realisation*, not information.
2. **Advice without liquidity is cruelty.** Every HOLD recommendation must come bundled with a cash option (pledge credit), or it converts a financial problem into a psychological one.
3. **Five loss mechanisms, not one** — information asymmetry, liquidity coercion, fragmentation, physical loss, counterparty risk. Confusing them is why agri-tech fails.
4. **eNAM already exists.** You are a *decision layer*, not a venue. Never say "we're building another eNAM."
5. **Arhatiya resistance.** Co-opt, don't eliminate — a commission agent registers as a verified buyer/aggregator and competes on reliability.
6. **Payment default is the #1 trust killer.** Escrow before dispatch; "funds in escrow before dispatch" is a hard rule.
7. **Quality is subjective.** The grader must be allowed to *abstain* ("cannot grade — get an assay"), never guess.
8. **Voice-first, not app-only.** Every money action must be completable by voice/read-back.
9. **Herd effect** — if everyone holds then dumps on day 14, you create the crash you predicted. Stagger recommendation dates.
10. **The pledge-credit guardrail is non-negotiable:** never offer credit unless expected net gain > finance cost, and repayment is auto-deducted from sale so the loan can't roll over.

### 1b. Blind spots specific to *your* situation (no pre-work, 1 week, college judges)

11. **"Real data" is a time trap.** Agmarknet is sparse, messy, and needs a hand-built canonical mapping — the playbook itself admits this repeatedly. Two days of scraping = demo dies. **Seed a small, clean, realistic onion dataset** and treat live ingestion as a Week-2 stretch.
12. **You cannot produce "real" ML in a week — don't fake it.** No honest LightGBM + conformal + Diebold-Mariano in 7 days without pre-work. A fabricated p-value is *worse* than a simpler honest model. Use a **transparent heuristic + quantile bands**, precompute it offline, and say what's simulated.
13. **Live ML serving is a demo-fatal dependency.** If the forecast needs a running Python process at demo time, it will be the thing that crashes. **Precompute forecasts to JSON, bake them into the app.** No model process at demo time.
14. **Every external dep is a network/credential risk.** Supabase, Bhashini, UPI, e-NWR all fail without a key or wifi. Use **SQLite + cached TTS audio + simulated escrow**, toggled by one env var (REAL vs MOCK).
15. **Six people on one repo = merge hell + scope drift.** Assign *disjoint file ownership*, freeze the schema at Day 4, and appoint one person with feature-freeze authority (the playbook's rule: no new feature after hour 28).
16. **The temptation to build all 14 requirements shallowly.** Your earlier sprint plans literally do this — a checkbox table of 14 features. That's the anti-pattern. **Deep on 3 pillars, shallow on the rest** is what wins.
17. **Overclaiming loses the room.** College judges include people who know agri economics. "Here's what's real, here's what's mocked" reads as maturity; implying everything works reads as inexperience.
18. **The cotton-vs-onion internal conflict.** Two of your docs disagree. Resolve it *now* (onion), so nobody debates it at Day 3.

---

## 2. What "80% of requirements" actually means (do not build a checkbox dashboard)

The problem statement has **14 explicit capabilities + 7 outcomes**. In 1 week you cannot do all 14 deeply. The playbook's own instruction: *"if your timeline is shorter, cut scope from Pillars 3 and 4, never from Pillars 1, 2 and 5."*

**Deep (the argument — Pillars 1, 2, 5):**
- Multi-mandi onion price board + 14-day forecast with uncertainty bands
- Sale-window recommendation (HOLD / SELL NOW / SELL ELSEWHERE / HOLD_WITH_PLEDGE) with net-of-cost arithmetic and a Marathi reason
- The pledge-credit "GET MONEY TODAY" flow (simulated, clearly labelled)
- **The Realisation Ledger** — per-transaction gain vs. counterfactual baseline + a backtested rupees-per-quintal number (G)

**Working-but-shallow (tick the other requirements with a seeded, walkable flow — Pillars 3, 4):**
- Lot creation + rule-based grading (grade A/B/C/D with a simple attribute form)
- FPO pooling with the grade-weighted fair-split ledger
- Buyer directory with verification tiers + reliability facts
- Matching (weighted score: price × grade × distance × reliability)
- Digital offer / accept / counter state machine
- Escrow state machine (CREATED → FUNDED → … → RELEASED | DISPUTED)
- Payment tracking + dispute/grievance flow
- Transport + storage directory (seeded rate cards)

**Honestly deferred to Week 2+ (say so):**
- Real CV grading, real payment rails, WhatsApp/IVR, native app, real ML depth, real e-NWR/AgriStack/Beckn integration.

---

## 3. Architecture (demo-first, precomputed ML, one deployable)

The single most important architectural decision: **the ML is a data artifact, not a service.**

### 3.1 Topology

```
farmer surface (Next.js, phone viewport)      buyer + public dashboard (Next.js)
        │                                              │
        └──────────────┬───────────────────────────────┘
                       ▼
        Next.js API routes (single app, TypeScript)
        ├── /api/prices, /api/forecast (reads baked JSON)
        ├── /api/window (M12 arithmetic)
        ├── /api/lots, /api/offers, /api/matches
        ├── /api/escrow, /api/disputes, /api/payments
        ├── /api/credit/quote (simulated pledge)
        └── /api/ledger (hash-chained, public district endpoint)
                       │
                       ▼
        SQLite via Prisma (Postgres-compatible schema)
        + baked forecast/backtest artifacts (JSON, committed to repo)
```

### 3.2 The two artifacts produced offline (Python, run once, committed)

`ml/forecast.py` and `ml/backtest.py` run **offline** and emit:
- `web/lib/data/artifacts/forecast_onion_lasalgaon.json` — `{mandi, commodity, run_date, points:[{h, p10, p50, p90}], drivers[], regime}`
- `web/lib/data/artifacts/backtest.json` — `{G_per_qtl, baseline_method, coverage, chart_data}`
- `web/lib/data/artifacts/precedents.json` — 3 nearest-neighbour past cases for the "why" panel

The Next.js app **imports** these at startup. No model, no Python, no network at demo time.

### 3.3 Model for Week 1 (honest, not fake)

- **Forecast:** seasonal-naive baseline + rolling quantiles for the bands. Simple, defensible, and honestly *labelled* "heuristic". Optionally a LightGBM quantile model if the ML owner finishes early — but the demo does not depend on it.
- **Sale-window (M12):** pure arithmetic over the forecast distribution + cost inputs (storage, transport, mandi fee, finance), with the farmer's cash-need constraint. Lives in TypeScript (`lib/window.ts`). This is the centrepiece and it's *not* ML — it's an auditable expected-utility calc, which is a strength.
- **Backtest G:** run the M12 rule over 2–3 years of the seeded onion series vs. a sell-on-harvest baseline. Report the honest number (even if modest — a small real number beats a big invented one).

### 3.4 Tech stack

| Layer | Choice | Why |
|---|---|---|
| App | Next.js 14 + TypeScript + Tailwind + shadcn/ui | One repo, API routes built-in, fastest to ship, demos well |
| DB | SQLite via Prisma (Postgres-compatible) | Zero external dep, fully offline, swap to Postgres+TimescaleDB later |
| ML | Python (pandas/numpy) — offline only | Produces artifacts; not a runtime dependency |
| Charts | Recharts | React-native, fast |
| Voice | Pre-generated Marathi TTS audio files (cached, committed) | No network/API key at demo |
| Deploy | Vercel for the live link; **demo runs on localhost with wifi off** | Network independence is the point |

---

## 4. Six-person division (maps to the demo critical path, not 14 features)

The playbook's R1–R6 is right in spirit; I'm restructuring it for *no pre-work + 1 week + demo-critical-path*.

| # | Role | Owns | Must NOT own |
|---|------|------|--------------|
| **P0** | **Decision Core (ML + backtest)** — strongest technical person | Offline forecast + bands, M12 spec, the backtested G number, the "why" precedents. The technical-credibility anchor. | Any UI. Cannot be split. |
| **P1** | **Backend / Data** | API routes, SQLite schema, seeded onion dataset, hash-chain ledger, buyer reliability score, escrow FSM, dispute flow. | The farmer app UI. |
| **P2** | **Farmer surface** | The sale-window screen (centrepiece), lot creation, Marathi "listen" toggle, the GET-MONEY-TODAY pledge flow, phone viewport. | Backend logic. |
| **P3** | **Buyer + Public surfaces + design** | Buyer dashboard, public district Realisation Ledger dashboard, admin view, the design system/visual polish. | Nothing else — this is what judges see first. |
| **P4** | **Pitch / Story / Field / Demo** | The narrative, PS-coverage table, 2 field touchpoints (an FPO/warehouse call or message = evidence), the 8-min demo script, deck, fallback video, risk Q&A. **Starts Day 1, never pulled onto the build.** | Tickets. This is half the score. |
| **P5** | **Integration / Lead / Floater** | Repo + one-command startup, offline hardening, schema-freeze enforcement, integration of Flow A + Flow B, absorbs whatever is behind. | A single feature that blocks others. |

**The most common 6-person mistake:** five coders + one "presentation person" at the end. P4 must start Day 1 and stay off the critical path — a working prototype with a farmer's name in it beats a rougher one with a story.

---

## 5. Roadmap

### Week 1 (internal hackathon) — ship the demo

| Day | Focus | Exit criteria (checkable) |
|---|---|---|
| **0–1** | Freeze scope (onion/Nashik). Scaffold repo. SQLite schema. Seed data. Offline forecast + backtest v1 → artifacts. API skeleton. | `pnpm dev` runs from clean clone. `G_per_qtl` number exists. Seed data loads. |
| **2–3** | Farmer surface (sale-window + pledge flow). Backend Pillars 3/4/5 (lots, offers, escrow, disputes, ledger). | Flow A (hold-or-sell) renders a recommendation from a real baked forecast. |
| **4** | Buyer dashboard + public ledger dashboard. Integrate Flow A and Flow B end-to-end. **Freeze schema.** | Lot → grade → pool → match → offer → escrow → release → ledger is walkable. |
| **5** | Marathi TTS cached. Offline mode verified (wifi off). Empty/error states. Seed 6 months of transaction history. PS-coverage table. | Every screen works with network disabled. |
| **6** | Full dry run (timed, recorded). Fallback video. Deck. **Feature freeze.** | 8-min run completes without a crash. |
| **7** | Buffer + 2–3 more rehearsals. | Every team member can run the demo alone. |

### Week 2 (if shortlisted) → 2–4 weeks (full project)

| Phase | Scope | Gate |
|---|---|---|
| **W2 — Real data** | Agmarknet ingester + canonical onion mapping + imputation with flags + data-quality report (it's a slide). | 3+ years × 10 Nashik mandis loaded. |
| **W2 — Real ML** | LightGBM quantile forecast beating persistence (MASE < 1), conformal coverage measured, **real** Diebold-Mariano p-value. | Coverage within 5 points of nominal. |
| **W3 — Channels + transaction** | WhatsApp + IVR voice, PWA/mobile, CV grading assist, real escrow/UPI integration, PM-AASHA proof export. | Money flow walkable on voice alone. |
| **W4 — Integrations + scale** | eNAM, e-NWR, AgriStack, ONDC/Beckn as the "decision layer on DPI" story; soybean/cotton expansion config. | Department dashboard used in the pitch as a policy tool. |

---

## 6. The demo script (8 min, aligned to the thesis)

1. **0:00–0:45 — one named farmer.** "Ramesh Pawar, 1.2 acres onion near Pimpalgaon. Sold 42 quintals at ₹890 on harvest day; 12 days later the same mandi was ₹1,340 — ₹18,900 he didn't get. He knew prices might rise; he owed a moneylender on Friday."
2. **0:45–1:30 — the insight.** "This isn't an information problem — Ramesh *knew*. The binding constraint is he couldn't afford to wait. So the product isn't a dashboard, it's a waiting machine."
3. **1:30–2:15 — five mechanisms → five pillars.** One slide, fast.
4. **2:15–4:30 — live demo part 1.** Sale-window screen: forecast *with the band*, HOLD 12 days, expected +₹3,700 / worst case −₹2,500, confidence "7 of 10". Play the Marathi audio. Then: "he still needs money Friday" → GET MONEY TODAY (warehouse, e-NWR, ₹38,000 today, ₹940 interest). **This is the moment you win.**
5. **4:30–6:00 — live demo part 2.** Create lot → grade → pool with fair-split table → two verified buyers ("pays in 3 days, never renegotiated") → accept → escrow funds. Mention dispute flow in one sentence.
6. **6:00–6:45 — the proof.** Realisation Ledger: per-transaction gain vs. counterfactual, then the backtest — "over 3 years of onion data, +₹G per quintal net of storage and finance costs vs. sell-on-harvest."
7. **6:45–7:30 — honesty + scale.** What's real vs. mocked, expansion path (onion → soybean → cotton → tomato → tur), integration story (eNAM, e-NWR, Bhashini, ONDC, AgriStack). "We don't replace government infrastructure — we're the decision layer on top of it."
8. **7:30–8:00 — close on the person.** "Ramesh doesn't need to be told the price. He needs to be able to wait for it. That's what we built."

---

## 7. Risks + the pre-written answers (bring to the room)

| Judge's question | Answer |
|---|---|
| "eNAM already exists" | "eNAM is a venue; we're a decision layer. By its own docs its trade is mostly intra-market — a Yavatmal farmer still can't reach Nagpur. We answer three questions eNAM doesn't: sell now or in 12 days, how do I get cash to wait, and which buyer will actually pay me." |
| "How do you get farmers to adopt?" | "Not one-by-one — through FPOs and state extension machinery. An FPO chairperson who trusts the fair-split ledger brings 60 farmers at once. First interaction is a phone call in Marathi, no app." |
| "What if the forecast is wrong?" | "It will be, and we designed for that — calibrated bands not point forecasts, downside shown before consent, advice suspended during policy shocks, and we publish our own hit rate. And when we say HOLD we also provide credit, so the farmer isn't cash-exposed to our error." |
| "Is this just Ninjacart?" | "No — they're a buyer, and their margin is the spread we exist to shrink. We're neutral and never take a position on the produce. Arya.ag is the closest analogue, but they're a financier with a platform; we're a state-scoped public market layer with finance as an integration." |
| "Who pays?" | "Buyer-side transaction fee only, never the farmer; or state funding as market infrastructure; or lender-side origination on pledge finance. A farmer paying for price info is the same asymmetry in a new coat." |

---

## 8. Verification (how we know it's done)

1. `pnpm dev` works from a clean clone on two machines.
2. Forecast API returns real onion numbers for Lasalgaon **with uncertainty bands**.
3. A recommendation renders on a phone from a baked forecast.
4. Flow A and Flow B walkable end-to-end, even if ugly.
5. The headline G number appears **in the product**, not just on a slide.
6. Demo survives with wifi physically off (including the Marathi audio).
7. Every claim in the deck is demonstrable in the product; everything simulated is labelled.
8. 8-minute run completes without a crash; fallback video queued.

---

## 9. Anti-goals (explicitly NOT building)

- No live ML serving, no blockchain (hash-chain + optional daily notarisation instead), no wallet, no native app, no pan-India, no all-crops, no eliminating the arhatiya, no taking a position on produce, no selling farmer data, no fabricating the G number.
