# MANDI-SETU — Pillars 1/2/5 Weekend Prototype (Pitch-Ready MVP)

> **Scope:** Pillar 1 (Price Intelligence) + Pillar 2 (Sale Window + Pledge Credit) + Pillar 5 (Realisation Ledger) — only.
> **Wedge:** Onion, Lasalgaon anchor, Nashik–Ahmednagar–Pune belt.
> **Timeline:** Friday → Sunday evening (≈3 days, 6 people).
> **Goal:** A working, offline, pitch-ready prototype that proves *one* idea end-to-end: **the farmer can afford to wait, and we can prove they gained money.**

---

## 1. The one decision (read this first)

We are **not** building a dashboard. We are building a **waiting machine**:

> **forecast (P1) → decision + cash (P2) → proof (P5)**

The farmer's binding constraint is the *ability to wait*, not information. Every HOLD recommendation must come bundled with a cash option (pledge credit), and every trade must be recorded against a counterfactual baseline so the claim "the farmer earned more" is a *measured number*, not an adjective.

**Everything not in service of this loop is out of scope this weekend.** Lots, grading, matching, escrow, disputes, logistics — all deferred to Week 2. If it doesn't appear in Flow A (below), we don't build it.

### The single demo flow (Flow A) — this is the whole product

```
Farmer opens app → today's onion price (Lasalgaon + nearby mandis)
  → 14-day forecast WITH uncertainty bands
  → recommendation: HOLD 12 days (+₹3,700 expected / −₹2,500 worst case / 7-of-10 confidence)
  → "NEED MONEY NOW?" → pledge: store at warehouse, ₹38,000 today, ₹940 interest
  → farmer accepts → transaction logged
  → Realisation Ledger shows: actual ₹ vs counterfactual ₹ → +₹G/qtl gained
  → public district dashboard: median gain, total gain, forecast coverage
```

---

## 2. What we are actually building (crisp feature list)

### Pillar 1 — Price Intelligence
- Multi-mandi onion price board: Lasalgaon + 5–8 nearby mandis (Nashik, Pimpalgaon, Ahmednagar, Solapur, Pune, Vashi terminal).
- 30-day price history chart + today's modal/min/max + arrivals.
- **14-day forecast with p10/p50/p90 bands** (1/3/7/14 horizons) — never a bare point forecast.
- A plain-Marathi "why" panel: top 2–3 drivers (arrival trend, seasonal index, neighbour-mandi signal, rainfall anomaly) + 2–3 nearest-neighbour past cases ("prices rose this way in 2 of the last 3 years").

### Pillar 2 — Sale Window + Pledge Credit
- The **M12 engine** (arithmetic, not ML): converts forecast distribution + costs + farmer cash-need into an action: `SELL_NOW | SELL_ELSEWHERE | HOLD | HOLD_WITH_PLEDGE`.
- Output card: expected gain/qty, **worst-case**/qty, confidence, target date + market, plain-Marathi reason, and a `cash_today` figure.
- **GET MONEY TODAY** pledge flow (simulated, clearly labelled): eligible warehouse, LTV, rate, tenor, net cash today, interest, repayment auto-deducted at sale.
- The **guardrail in code**: never offer credit unless expected net gain > finance cost.

### Pillar 5 — Realisation Ledger
- Per-transaction record: `realised_net` vs `counterfactual` (same-day default-mandi modal price) → `realised_gain_per_qtl`.
- Append-only **SHA-256 hash chain** for tamper-evidence + farmer-downloadable statement.
- **Public district dashboard**: median gain/qtl, total gain, forecast coverage, follow rate.
- A **backtested G** (₹/quintal) number — computed offline over the seeded series, shown in the product.

---

## 3. System architecture

### 3.1 The governing principle: **ML is a data artifact, not a service**

Live ML serving is the #1 demo-fatal dependency. In a weekend there is no honest way to run a trained model at request time and survive. So:

> **All forecasting and backtesting runs offline in Python, emits JSON, and the JSON is committed to the repo and imported by the app.** At demo time there is no Python process, no model, no network.

### 3.2 Topology

```
OFFLINE (Python, run once, committed)        RUNTIME (Next.js, single app)
─────────────────────────────────────       ────────────────────────────────
generate_seed.py ──► seed.sqlite             / farmer sale-window screen
forecast.py      ──► forecast.json  ──┐      / price board
backtest.py      ──► backtest.json    ├────► / ledger dashboard
precedents.py    ──► precedents.json ─┘       / farmer statement
                                              API routes (TS):
                                                /api/prices
                                                /api/forecast
                                                /api/window   ← M12 engine (TS)
                                                /api/credit/quote
                                                /api/ledger
                                              SQLite via Prisma (Postgres-compatible)
                                              Marathi TTS = cached .mp3, committed
```

### 3.3 Data flow (the three steps)

1. **Ingest → store.** `generate_seed.py` writes `price_obs` (3 years, ~9 mandis, onion) + `farmer` + `market` + seeded `transaction` history into SQLite. *(Swap to real Agmarknet in Week 2 — the schema doesn't change.)*
2. **Forecast → artifact.** `forecast.py` reads `price_obs`, computes quantile bands + drivers + precedents, writes `forecast.json` / `precedents.json`.
3. **Backtest → G.** `backtest.py` runs the M12 rule over the series vs sell-on-harvest baseline, writes `backtest.json` (contains the G number + coverage + chart data).

The Next.js app imports these artifacts at build/startup. `/api/window` does the live arithmetic per farmer input.

### 3.4 Component responsibilities

| Component | Language | Job | Output |
|---|---|---|---|
| `ml/generate_seed.py` | Python | Generate realistic onion series + seed DB | `seed.sqlite` |
| `ml/forecast.py` | Python | Quantile bands + drivers + precedents | `forecast.json`, `precedents.json` |
| `ml/backtest.py` | Python | Run M12 over history → G + coverage | `backtest.json` |
| `lib/window.ts` | TypeScript | M12 decision arithmetic (the crown jewel) | recommendation object |
| `lib/ledger.ts` | TypeScript | SHA-256 hash chain + counterfactual calc | ledger entries |
| `app/**` | Next.js | Farmer screen, price board, ledger dashboard | UI |
| `app/api/**` | Next.js | Read artifacts, serve M12 + ledger | REST |

---

## 4. Data model (SQLite via Prisma, Postgres-compatible)

Keep it to **exactly what the three pillars need**. No lots/offers/escrow tables yet.

```sql
-- dims
farmer(farmer_id PK, name, name_mr, village, district)
market(market_id PK, name, name_mr, district, lat, lon, is_anchor)
commodity(commodity_id PK, name, name_mr, storability_days)

-- facts (time series)
price_obs(
  market_id FK, commodity_id FK, obs_date,
  min_p, max_p, modal_p, arrivals_qtl, source
)

-- forecast artifact (imported from JSON, but mirrored for audit)
forecast(
  forecast_id PK, market_id, commodity_id, run_date,
  horizon_days, p10, p50, p90, model_version
)

-- recommendation (every decision persisted + auditable)
recommendation(
  rec_id PK, farmer_id, commodity_id, qty_qtl, harvest_date,
  action, target_date, target_market_id,
  exp_gain_per_qtl, downside_per_qtl, confidence,
  cash_today, reason_mr, model_version, features_json
)

-- realisation (the proof layer — hash-chained)
realisation(
  txn_id PK, farmer_id, commodity_id, qty_qtl,
  sold_price, sold_date, counterfactual_price, baseline_method,
  realised_gain_per_qtl, net_amt,
  prev_hash, entry_hash
)
```

Two details that matter to a technical judge:
- `recommendation.features_json` + `model_version` → any advice can be reproduced exactly (auditability).
- `realisation.baseline_method` → the counterfactual is recomputable, not asserted.

---

## 5. The M12 sale-window engine (spec, TypeScript)

This is the intellectual core. It is **arithmetic over a distribution**, not a learned model — say that out loud in the pitch; it reads as judgement, not weakness.

### 5.1 Inputs

```
qty_qtl                      Q
forecast.points[]            {h, p10, p50, p90} per horizon per market
spoilage(h)                  fraction lost by day h (agronomic curve, cited)
C_st(h)                      storage cost = rate * h
C_tr(m), C_mk(m)             transport, mandi fee + commission
C_fi(h)                      finance cost of pledge advance over h days
cash_need, cash_need_by      farmer liquidity constraint
rho                          risk preference ∈ [0,1] (0 = best average, 1 = safest)
```

### 5.2 Decision arithmetic

```
baseline  net_now = P(default,0) * Q * (1 - spoilage(0)) - C_tr(default) - C_mk(default)

for each (h, m):
    gross_p50 = F(h,m).p50 * Q * (1 - spoilage(h))
    gross_p10 = F(h,m).p10 * Q * (1 - spoilage(h))
    costs     = C_st(h) + C_tr(m) + C_mk(m) + (advance_needed ? C_fi(h) : 0)
    net_p50   = gross_p50 - costs
    net_p10   = gross_p10 - costs
    utility   = (1 - rho) * net_p50 + rho * net_p10   // explicit risk aversion

    cash constraint:
      if cash_need > 0 and cash_need_by < today + h:
          advance = credit_quote(lot_value = gross_p50, tenor = h)
          if advance.net_cash_today >= cash_need: advance_needed = true
          else: mark INFEASIBLE unless a partial-sale SPLIT covers it

best = argmax utility over feasible (h, m)

exp_gain_per_qtl = (best.utility - net_now) / Q
downside_per_qtl = (best.net_p10 - net_now) / Q

action:
  if best.utility - net_now < threshold(Q): SELL_NOW          // never advise a trip for ₹200
  elif best.h == 0 and best.m != default:     SELL_ELSEWHERE
  elif advance_needed:                        HOLD_WITH_PLEDGE
  else:                                       HOLD
```

### 5.3 The three details that impress a judge

1. **`rho`** — we *ask* the farmer risk preference; the maths changes with it.
2. **SPLIT** — sell part now to cover cash, hold the rest (what a smart farmer already does informally, made computable).
3. **`threshold(Q)`** — we refuse advice whose expected gain is smaller than the hassle.

### 5.4 The pledge guardrail (in code, not terms)

```
credit_quote() returns null (no offer) unless:
  expected_net_gain > finance_cost
repayment is auto-deducted from sale proceeds → loan cannot roll over
```

---

## 6. The forecast + backtest artifacts (honest, no fake ML)

### 6.1 Forecast method (weekend-honest)

- **p50** = seasonal-naive (same period last year, level-adjusted) + momentum term from last 7–14 days.
- **p10/p90** = p50 ∓ `k(h) * rolling_volatility(h)`, with `k` chosen so empirical coverage ≈ 80–90% on a held-out slice.
- **drivers[]** = top contributors: arrival trend (rising arrivals → downward pressure), seasonal index, neighbour-mandi signal, rainfall anomaly flag.
- **regime** = `normal | shift` (a hand-built onion policy-event flag — export ban/MEP — widens the band).
- **precedents[]** = 2–3 historical cases most similar to now (nearest-neighbour over the feature vector).

**Output shape (`forecast.json`):**
```json
{
  "market_id": "lasalgaon", "commodity_id": "onion", "run_date": "2026-09-04",
  "regime": "normal", "model_version": "seasonal-naive-v1",
  "points": [ { "h": 1, "p10": 1210, "p50": 1240, "p90": 1270 },
              { "h": 14, "p10": 1180, "p50": 1390, "p90": 1500 } ],
  "drivers": [ { "name": "arrival_trend", "direction": "down", "magnitude": 0.6 } ],
  "coverage_last_90d": { "nominal_80": 80, "empirical_80": 78 }
}
```

**If the ML owner finishes early**, drop in a LightGBM quantile model — but the demo does not depend on it. The honest label is the asset.

### 6.2 Backtest → the G number

Run the M12 rule over the seeded 3-year onion series vs a **sell-on-harvest-day baseline** (the counterfactual). Report:

- `G_per_qtl` = median realised gain per quintal, **net of storage + finance costs**.
- coverage table (empirical vs nominal).
- a chart (already rendered in the product).

**The rule:** G must come from this backtest. A small real number beats a big invented one. If G is disappointing, report the honest G — it's still *yours* and defensible.

---

## 7. The Realisation Ledger (proof layer)

### 7.1 Counterfactual baseline

```
counterfactual_price = same-day modal price at the farmer's DEFAULT mandi
baseline_method = "sell-on-harvest-day default-mandi modal"   // stored, recomputable
realised_gain_per_qtl = sold_price - counterfactual_price
                       - (storage + finance costs, for held lots)
```

### 7.2 Hash chain

```
entry_hash = SHA256(prev_hash || txn_id || farmer_id || commodity_id || qty
                    || sold_price || counterfactual_price || baseline_method
                    || timestamp)
```

Properties: tamper-evidence, farmer-verifiable (download + recompute), cheap (one table, no consensus layer). Say this explicitly — *"we considered a blockchain and rejected it; a hash chain + periodic notarisation gives us tamper-evidence at a fraction of the cost."*

### 7.3 Public district dashboard

`/ledger` shows: median gain/qtl, total ₹ gained, forecast coverage, follow rate, and a live-updating entry feed. This is the **policy instrument** — the thing the government department can actually use.

---

## 8. Tech stack

| Layer | Choice | Why (weekend reasoning) |
|---|---|---|
| App + API | Next.js 14 + TypeScript | One repo, API routes built-in, fast, demos well |
| UI | Tailwind + shadcn/ui | Copy-paste components, no custom CSS |
| Charts | Recharts | React-native, fast |
| DB | SQLite via Prisma | **Zero external dep**, fully offline; swap to Postgres+TimescaleDB in Week 2 |
| ML | Python (pandas/numpy) — **offline only** | Produces JSON artifacts; never a runtime dep |
| Voice | Pre-generated Marathi TTS `.mp3`, committed | No network/API key at demo |
| Deploy | Vercel for the live link; **demo runs localhost with wifi off** | Network independence is the point |

---

## 9. Work division (6 people, precise ownership)

The rule from the playbook, restated: **nobody on the demo-critical path may be pulled onto a second critical path.** Disjoint file ownership prevents merge hell.

| # | Role | Owns (files/deliverables) | Must NOT own |
|---|------|---------------------------|--------------|
| **P0** | **ML + backtest (strongest technical)** | `ml/*.py` → seed data, `forecast.json`, `precedents.json`, `backtest.json`, **the G number** | Any UI. Cannot be split. |
| **P1** | **Backend + M12 + ledger** | `lib/window.ts`, `lib/ledger.ts`, `app/api/**`, Prisma schema, seed loading | Farmer screen UI |
| **P2** | **Farmer surface** | `/` (sale-window screen), `/prices`, pledge flow UI, Marathi "listen" toggle, phone viewport | Backend logic |
| **P3** | **Ledger dashboard + design** | `/ledger` (district dashboard), farmer statement view, design system, visual polish | Nothing else — judges see this first |
| **P4** | **Pitch / story / demo** | Narrative, PS-coverage table, demo script, fallback video, risk Q&A, 1–2 field touchpoints (FPO/warehouse call = evidence) | Tickets. Starts Day 1, never pulled onto build. |
| **P5** | **Integration / lead / floater** | Repo scaffold, one-command startup, offline hardening, schema-freeze enforcement, Flow A integration, absorbs overflow | Any single feature that blocks others |

**The most common 6-person mistake:** five coders + one "presentation person" at the end. P4 starts Friday morning and stays off the build. A working prototype *with a farmer's name in it* beats a rougher one with a story.

---

## 10. Day-by-day build plan (Friday → Sunday evening)

### Day 1 — Friday: freeze, scaffold, data, G number exists

| Time | Owner | Deliverable |
|---|---|---|
| Morning | All | Scope freeze (onion/Lasalgaon, Pillars 1/2/5 only). Sign off the Flow A narrative. |
| Morning | P5 | Repo scaffold, `pnpm dev` running, Prisma schema, `.env` with `DATA_MODE=seed\|real`. |
| Midday | P0 | `generate_seed.py` → realistic onion series for 9 mandis (3 yrs) + seed transactions. |
| Midday | P1 | API skeleton: `/api/prices`, `/api/forecast`, `/api/window`, `/api/ledger` (stubs). |
| Afternoon | P0 | `forecast.py` + `precedents.py` → artifacts. `backtest.py` → **G number is known by Friday night.** |
| Afternoon | P1 | `lib/window.ts` (M12) implemented against the spec; `lib/ledger.ts` hash chain. |
| Afternoon | P2 | Sale-window screen shell (static, consuming stub data). |
| Afternoon | P3 | Ledger dashboard shell + design tokens. |
| Evening | P4 | Demo script v1, PS-coverage table, field touchpoint outreach. |
| Evening | P5 | Wire artifacts → API → frontend. **Checkpoint: `pnpm dev` runs from clean clone; G number exists; seed loads.** |

### Day 2 — Saturday: build the loop end-to-end

| Time | Owner | Deliverable |
|---|---|---|
| Morning | P1 | Finish M12 + credit quote + recommendation persistence. |
| Morning | P0 | Refine forecast bands (coverage check); hand-build onion policy-event flag. |
| Morning | P2 | Sale-window screen: forecast band chart, HOLD/SELL card, worst-case, confidence "7-of-10", GET MONEY TODAY flow. |
| Morning | P3 | Ledger dashboard: median gain, total ₹, coverage, live feed. |
| Afternoon | P5 | Integrate Flow A end-to-end: farmer input → recommendation → accept → ledger append. |
| Afternoon | P4 | Rehearse narration against whatever exists. |
| Evening | P5 | **Schema freeze.** **Checkpoint: Flow A walkable; a recommendation renders from a real baked forecast; ledger renders seeded history.** |

### Day 3 — Sunday: Marathi, offline, polish, pitch

| Time | Owner | Deliverable |
|---|---|---|
| Morning | P4 | Pre-generate Marathi TTS `.mp3` for every demo line; commit. |
| Morning | P2 | Wire TTS; "listen in Marathi" plays the cached audio. |
| Morning | P3 | Polish the three screens judges will see; add empty/error states. |
| Midday | P5 | **Offline hardening:** verify wifi-off, error states, `DATA_MODE` toggle. |
| Afternoon | All | Full dry run (timed, recorded). Fix breakages. **Feature freeze.** |
| Afternoon | P5 | Fallback demo video (subtitled, two laptops + one phone). |
| Evening | P4 | Deck, one-pager, risk answers. 2–3 more rehearsals. |

**Sunday evening exit criteria:**
- `pnpm dev` runs from clean clone.
- Forecast returns real onion numbers **with bands**.
- Recommendation renders from a baked forecast.
- Ledger renders and shows the G number.
- Demo survives wifi-off (including Marathi audio).
- 8-min run completes without a crash; fallback video queued.
- Every team member can run the demo alone.

---

## 11. Pitch-ready checklist

- [ ] G number appears **in the product**, not just a slide.
- [ ] Every forecast shown with its uncertainty band.
- [ ] HOLD recommendation always shows the cash option below it.
- [ ] Worst-case shown with equal weight to expected.
- [ ] Everything simulated is labelled "simulated"; real data labelled real.
- [ ] One named farmer + one real FPO/warehouse touchpoint (a quote or a rate card = evidence).
- [ ] The 8-min script rehearsed ≥3× (once with the demo broken on purpose).
- [ ] PS-coverage table (14 requirements → which pillar, status).
- [ ] Pre-written answers to the 5 killer questions (below).

---

## 12. Demo script (8 min)

1. **0:00–0:45 — one named farmer.** "Ramesh Pawar, 1.2 acres onion near Pimpalgaon. Sold 42 quintals at ₹890 on harvest day; 12 days later the same mandi was ₹1,340 — ₹18,900 he didn't get. He knew prices might rise; he owed a moneylender on Friday."
2. **0:45–1:30 — the insight.** "This isn't an information problem — Ramesh *knew*. The binding constraint is he couldn't afford to wait. So we built a waiting machine, not a dashboard."
3. **1:30–2:15 — five mechanisms → three pillars (this weekend).** One slide, fast.
4. **2:15–4:30 — live demo part 1.** Sale-window: forecast *with band*, HOLD 12 days, +₹3,700 / −₹2,500 / 7-of-10. Play Marathi audio. Then "he still needs money Friday" → GET MONEY TODAY (₹38,000 today, ₹940 interest). **The moment you win.**
5. **4:30–6:00 — live demo part 2.** Show the transaction being logged → Realisation Ledger: actual vs counterfactual → +₹G/qtl.
6. **6:00–6:45 — the proof.** The backtest: "over 3 years of onion data, +₹G/qtl net of storage and finance costs vs sell-on-harvest."
7. **6:45–7:30 — honesty + scale.** What's real vs mocked; expansion (soybean → cotton → tomato → tur); integrations (eNAM, e-NWR, Bhashini, ONDC, AgriStack). "We don't replace government infrastructure — we're the decision layer on top of it."
8. **7:30–8:00 — close on the person.** "Ramesh doesn't need to be told the price. He needs to be able to wait for it. That's what we built."

---

## 13. The five killer questions (pre-written)

| Question | Answer |
|---|---|
| "eNAM already exists" | "eNAM is a venue; we're a decision layer. Its trade is mostly intra-market — a Yavatmal farmer still can't reach Nagpur. We answer three questions eNAM doesn't: sell now or in 12 days, how do I get cash to wait, and did I actually gain." |
| "How do you get farmers to adopt?" | "Through FPOs and state extension — an FPO chairperson who trusts the fair-split ledger brings 60 farmers at once. First interaction is a Marathi phone call, no app." |
| "What if the forecast is wrong?" | "It will be — we designed for that: calibrated bands not point forecasts, downside shown before consent, advice suspended during policy shocks, and we publish our hit rate. And when we say HOLD we also provide credit, so the farmer isn't cash-exposed to our error." |
| "Is this just Ninjacart?" | "No — they're a buyer; their margin is the spread we exist to shrink. We're neutral, never take a position on produce. Arya.ag is closest, but they're a financier with a platform; we're a state-scoped public market layer." |
| "Who pays?" | "Buyer-side fee only, never the farmer; or state funding as market infrastructure; or lender-side origination on pledge finance. A farmer paying for price info is the same asymmetry in a new coat." |

---

## 14. Anti-goals (explicitly NOT building this weekend)

- No lots, grading, matching, escrow, disputes, logistics (Pillars 3/4 → Week 2).
- No live ML serving, no blockchain (hash chain + notarisation instead), no wallet, no native app.
- No pan-India, no all-crops (onion only).
- No fabricating the G number.
- No taking a position on produce; no selling farmer data.
