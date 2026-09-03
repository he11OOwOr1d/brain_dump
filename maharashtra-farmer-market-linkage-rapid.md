# Maharashtra Farmer Market Linkage — Rapid Build Plan

> **Timeline**: 1 week for 80% MVP → 2-4 weeks for full project
> **Mode**: Hackathon-style rapid prototyping
> **Goal**: Demo-able product that showcases the full vision

---

## Revised Strategy: "Demo-First, Build-Second"

With 1 week to show 80% of requirements, we need to flip the approach:

**Original plan**: Build backend first, then UI, then scale
**Rapid plan**: Build a stunning demo that LOOKS complete, with real data where possible and smart mock data where not

**Key insight**: For a government hackathon/demo, a working prototype with:
- Real mandi price data (scraped or API)
- Beautiful visualizations
- Clear user flows
- Working matching logic

...will score higher than a "real" product with poor UX.

---

## Week 1 MVP: "Cotton Market Intelligence Platform"

### What We Build (Days 1-7)

| Day | Deliverable | Tech |
|-----|-------------|------|
| **Day 1** | Project setup, data pipeline, landing page | Next.js, Tailwind, Vercel |
| **Day 2** | Mandi price dashboard (real data) | eNAM scraping / mock data |
| **Day 3** | Price trends + sale window predictor | Chart.js, simple ML heuristic |
| **Day 4** | Buyer directory + FPO registration | Forms, local storage/Supabase |
| **Day 5** | Matching engine + recommendations | Algorithm + UI |
| **Day 6** | Mobile responsive + Marathi toggle | i18n, responsive design |
| **Day 7** | Polish, demo video, presentation | Loom, slide deck |

---

### Tech Stack (Optimized for Speed)

```
Frontend:    Next.js 14 (App Router) + Tailwind CSS + shadcn/ui
Backend:     Next.js API routes (no separate backend needed)
Database:    Supabase (free tier) or local JSON for MVP
Charts:      Recharts (React-native, beautiful)
Deployment:  Vercel (free, instant deploy)
Data:        eNAM public data + MKISAN scraping + mock fallback
Maps:        React Leaflet (free, no API key needed)
```

**Why this stack:**
- Next.js: One framework for frontend + backend, fastest to ship
- Tailwind + shadcn: Pre-built components, no custom CSS
- Supabase: Free PostgreSQL, auth, storage — no backend dev needed
- Vercel: Push to git → live in 30 seconds

---

### Feature Set for Week 1 MVP

#### 1. Landing Page (Day 1)
- Problem statement (from PS #26132)
- Solution overview with screenshots
- "View Live Demo" CTA
- Marathi/English toggle

#### 2. Mandi Price Dashboard (Day 2)
**Shows**: Real-time cotton prices from Maharashtra APMCs

| Feature | Implementation |
|---------|----------------|
| Price table | Top 10 APMCs by volume, sorted by price |
| Price cards | Min/Max/Modal price per quintal |
| Last updated | Timestamp (shows data freshness) |
| Data source | eNAM API or scraped from MKISAN |

**Data approach:**
- Try eNAM API first (if accessible)
- Fallback: Scrape from MKISAN portal (public data)
- Last resort: Realistic mock data based on actual cotton prices (₹5,800-6,500/quintal range)

#### 3. Price Trends + Sale Window (Day 3)
**Shows**: Historical prices + AI recommendation

| Feature | Implementation |
|---------|----------------|
| 30-day price chart | Line chart, APMC-wise comparison |
| Arrival volume chart | Bar chart, shows supply trends |
| Sale window signal | 🟢 SELL NOW / 🟡 HOLD / 🔴 WAIT |
| Recommendation text | "Based on arrival trends, prices may rise 5-8% in 7 days" |

**Sale window algorithm (simple heuristic):**
```
IF arrivals_decreasing AND demand_stable → SELL NOW (prices rising)
IF arrivals_increasing AND demand_stable → HOLD (prices falling)
IF arrivals_stable AND demand_increasing → SELL NOW
ELSE → HOLD
```

This is rule-based, not ML — but it's explainable and demo-able.

#### 4. Buyer Directory (Day 4)
**Shows**: Verified buyers with their requirements

| Field | Example |
|-------|---------|
| Buyer name | "Maharashtra Cotton Mills, Yavatmal" |
| Location | Yavatmal, Maharashtra |
| Cotton grade | Grade A (staple length >28mm) |
| Price offered | ₹6,200/quintal |
| Quantity needed | 500 quintals/week |
| Payment terms | 7 days after delivery |
| Contact | +91-XXXXX-XXXXX |

**Implementation:**
- Static list of 10-15 realistic buyers (based on real Maharashtra mills)
- Filter by location, grade, price
- "Contact Buyer" button (shows phone/email)

#### 5. FPO Registration (Day 4)
**Form fields:**
- FPO name
- District
- Contact person
- Phone number
- Cotton production (quintals/year)
- Current selling price
- Storage availability (yes/no)

**Implementation:**
- Form → Supabase database or local storage
- Shows success message
- Data used for matching

#### 6. Matching Engine (Day 5)
**Shows**: "Here are the best buyers for your cotton"

**Algorithm:**
```
1. Get FPO location (district)
2. Filter buyers within 100km
3. Match quality grade (if FPO specified)
4. Sort by price offered (highest first)
5. Show top 3 matches with reasoning
```

**Output:**
```
🎯 Top 3 Buyers for Your Cotton:

1. Maharashtra Cotton Mills, Yavatmal
   → ₹6,200/quintal (₹200 above market)
   → 15 km from your FPO
   → Needs 500 quintals/week
   → Payment in 7 days

2. Vidarbha Cotton Ginning, Wardha
   → ₹6,050/quintal
   → 45 km from your FPO
   → Needs 200 quintals/week
   → Payment in 14 days

3. [etc.]
```

#### 7. Lot Creation (Day 5)
**Form:**
- FPO name (dropdown if registered)
- Cotton quantity (quintals)
- Quality grade (Grade A/B/C)
- Storage location
- Expected price
- Available from (date)

**Output:**
- Creates a "lot" visible to buyers
- Shows on dashboard as "Active Lots"
- Buyers can express interest

#### 8. Admin Dashboard (Day 6)
**Shows:**
- Total FPOs registered
- Total lots created
- Total buyers on platform
- Recent activity
- Price alerts sent

---

### What We Cut for Week 1 (Save for Week 2-4)

| Feature | Why Cut | Add In |
|---------|---------|--------|
| WhatsApp bot | Too complex for 1 week | Week 2 |
| Real payment integration | Needs bank partnership | Week 3-4 |
| Logistics coordination | Needs transporter partnerships | Week 3-4 |
| Dispute resolution | Needs legal framework | Week 4 |
| Computer vision grading | Needs ML model training | Post-hackathon |
| IVR voice system | Needs telecom integration | Week 3 |
| Mobile app (native) | React Native takes 2+ weeks | Week 4 (PWA instead) |
| Real-time data pipeline | Needs robust scraping infra | Week 2 |

---

## Week 2-4: Full Project Enhancements

### Week 2: Data + Notifications
- Real eNAM API integration (if not done in Week 1)
- WhatsApp notifications via Twilio/WhatsApp Business API
- Email alerts (SendGrid/Resend)
- Daily price digest (automated)
- More crops (soybean, onion)

### Week 3: Transactions
- Digital offer acceptance (buyer offers → FPO accepts)
- Payment tracking dashboard
- Integration with eNAM payment system
- Basic escrow (partner with payment gateway)

### Week 4: Scale + Polish
- Marathi language full support
- Mobile PWA (installable web app)
- Admin panel for government users
- Analytics dashboard (user engagement, price improvements)
- Export to PDF/Excel

---

## Project Structure (Next.js)

```
maharashtra-farmer-market/
├── app/
│   ├── page.tsx                    # Landing page
│   ├── layout.tsx                  # Root layout
│   ├── globals.css                 # Tailwind styles
│   ├── (dashboard)/
│   │   ├── prices/
│   │   │   └── page.tsx            # Mandi price dashboard
│   │   ├── trends/
│   │   │   └── page.tsx            # Price trends + sale window
│   │   ├── buyers/
│   │   │   └── page.tsx            # Buyer directory
│   │   ├── matching/
│   │   │   └── page.tsx            # FPO-Buyer matching
│   │   ├── lots/
│   │   │   ├── page.tsx            # Active lots
│   │   │   └── create/
│   │   │       └── page.tsx        # Create lot form
│   │   └── register/
│   │       └── page.tsx            # FPO registration
│   ├── api/
│   │   ├── prices/
│   │   │   └── route.ts            # Price data API
│   │   ├── buyers/
│   │   │   └── route.ts            # Buyers API
│   │   ├── lots/
│   │   │   └── route.ts            # Lots CRUD
│   │   └── fpo/
│   │       └── route.ts            # FPO registration
│   └── admin/
│       └── page.tsx                # Admin dashboard
├── components/
│   ├── ui/                         # shadcn components
│   ├── price-card.tsx              # Price display card
│   ├── price-chart.tsx             # Price trend chart
│   ├── sale-window-badge.tsx       # SELL/HOLD signal
│   ├── buyer-card.tsx              # Buyer info card
│   ├── lot-card.tsx                # Lot display card
│   ├── matching-result.tsx         # Match display
│   ├── header.tsx                  # Navigation
│   └── footer.tsx                  # Footer
├── lib/
│   ├── data/
│   │   ├── prices.ts               # Price data (mock/real)
│   │   ├── buyers.ts               # Buyer data
│   │   └── apmc.ts                 # APMC master data
│   ├── utils/
│   │   ├── matching.ts             # Matching algorithm
│   │   ├── sale-window.ts          # Sale window logic
│   │   └── i18n.ts                 # Marathi translations
│   └── supabase.ts                 # Supabase client
├── types/
│   └── index.ts                    # TypeScript types
├── tailwind.config.ts
├── next.config.js
├── package.json
└── README.md
```

---

## Data Strategy (Week 1)

### Option A: Real Data (Preferred)
1. **eNAM API** — Check if public API exists for Maharashtra cotton prices
2. **MKISAN scraping** — Public portal, scrape daily prices
3. **APMC websites** — Some publish daily rates

### Option B: Mock Data (Fallback)
Create realistic mock data based on actual Maharashtra cotton prices:

```typescript
// Example: Realistic cotton price data
const mockPrices = [
  { apmc: "Yavatmal", min: 5800, max: 6400, modal: 6200, arrivals: 12500, date: "2026-09-03" },
  { apmc: "Wardha", min: 5750, max: 6350, modal: 6150, arrivals: 8200, date: "2026-09-03" },
  { apmc: "Akola", min: 5850, max: 6450, modal: 6250, arrivals: 15000, date: "2026-09-03" },
  // ... etc
]
```

**Key**: Use realistic numbers (actual cotton prices in Maharashtra range ₹5,800-6,500/quintal). This makes the demo credible.

---

## Demo Flow (For Judges)

**2-minute demo script:**

1. **Problem** (15 sec): "Maharashtra's 15M farmers sell cotton without knowing where prices are highest. They lose 10-15% of potential income."

2. **Solution overview** (15 sec): "We built a platform that aggregates mandi prices, predicts sale windows, and matches FPOs with verified buyers."

3. **Live demo** (90 sec):
   - Show landing page
   - Navigate to "Mandi Prices" → Show real-time cotton prices from 10 APMCs
   - Navigate to "Price Trends" → Show 30-day chart + "SELL NOW" recommendation
   - Navigate to "Buyer Directory" → Show 10 verified buyers with prices
   - Navigate to "Matching" → Enter FPO details → Show top 3 buyer matches
   - Show "Create Lot" → Fill form → Show lot created

4. **Impact** (15 sec): "Farmers using our platform can improve price realization by 8-12%. For a cotton FPO with 500 quintals, that's ₹2-3 lakh extra income per season."

5. **Scale** (15 sec): "Starting with cotton in Vidarbha, expanding to all Maharashtra crops by 2027."

---

## Team Roles (If Working in Team)

| Role | Responsibility | Days |
|------|----------------|------|
| **Frontend Dev** | All UI pages, components, charts | Day 1-6 |
| **Backend Dev** | API routes, data pipeline, database | Day 1-5 |
| **Data/ML** | Price data, sale window algorithm, matching | Day 2-5 |
| **Design/PM** | UX flow, demo script, presentation | Day 1, 6-7 |

**Solo developer**: Focus on frontend-first, use mock data, add backend later.

---

## Success Criteria (Week 1)

| Criteria | Target |
|----------|--------|
| **Working demo** | Live URL, all pages accessible |
| **Realistic data** | Cotton prices in realistic range |
| **User flow** | Complete flow from FPO registration → matching → lot creation |
| **Mobile responsive** | Works on phone (judges will check) |
| **Marathi support** | At least landing page + key labels in Marathi |
| **Presentation** | 2-min demo video + slide deck |

---

## Immediate Action Items (Start NOW)

1. **Create Next.js project**: `npx create-next-app@latest maharashtra-farmer-market`
2. **Install dependencies**: `npm install recharts @supabase/supabase-js clsx tailwind-merge`
3. **Set up Tailwind + shadcn**: `npx shadcn-ui@latest init`
4. **Create data file**: `lib/data/prices.ts` with mock cotton prices
5. **Build landing page**: Problem + solution + CTA
6. **Build price dashboard**: Table + cards with mock data
7. **Iterate**: Add features one by one

---

## Summary

| Phase | Timeline | Deliverable |
|-------|----------|-------------|
| **Week 1 MVP** | Days 1-7 | Demo-able web app with prices, trends, matching, lots |
| **Week 2** | Days 8-14 | Real data integration, WhatsApp alerts, more crops |
| **Week 3** | Days 15-21 | Transaction features, payment tracking |
| **Week 4** | Days 22-30 | Polish, Marathi support, admin panel, PWA |

**The 80/20 rule**: 20% of features (price dashboard + matching) deliver 80% of demo value. Focus there first.
