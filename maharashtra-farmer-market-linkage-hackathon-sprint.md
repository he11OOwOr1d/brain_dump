# Maharashtra Farmer Market Linkage — Hackathon Sprint Plan

> **Context**: College internal hackathon, university judges, working prototype in 1 week
> **Target**: 80% of problem statement requirements
> **Team**: 6 people
> **Data**: Real eNAM/APMC data available
> **Deliverable**: Working demo + prototype

---

## 1. What "80% of Requirements" Actually Means

The problem statement has 14 requirements. Here's what we build in 1 week vs. what we defer:

### BUILD IN WEEK 1 (The 80% — Demo Core)

| # | Requirement | What We Build | Demo Value |
|---|-------------|---------------|------------|
| 1 | **Aggregate mandi prices** | Live price dashboard from 10+ APMCs | ⭐ High — visual impact |
| 2 | **Buyer demand signals** | Buyer posts demand, visible to farmers | ⭐ High — two-sided marketplace |
| 3 | **Quality requirements** | Quality specs shown per commodity | Medium |
| 4 | **Arrival volumes** | Arrival data visualization | ⭐ High — price correlation |
| 5 | **Transport/storage options** | Directory of logistics providers | Medium |
| 6 | **Localized price trends** | Interactive charts, historical trends | ⭐ High — visual impact |
| 7 | **Sale window recommendations** | "Sell Now" / "Hold" prediction with % | ⭐ High — AI/tech showcase |
| 8 | **Match farmers/FPOs with buyers** | Matching engine + notification | ⭐ High — core value prop |
| 9 | **Lot creation** | FPO creates digital lot with quality details | ⭐ High — transaction flow |
| 10 | **Quality grading** | Grade selection + display per lot | Medium |
| 11 | **Digital offers** | Buyer makes offer on lot, farmer accepts | ⭐ High — transaction flow |
| 12 | **Logistics coordination** | Logistics booking (form + status) | Medium |
| 13 | **Payment tracking** | Payment status workflow (initiated → completed) | ⭐ High — trust element |
| 14 | **Dispute/grievance process** | Dispute filing + status tracking | Medium |

### DEFER TO WEEK 2 (Polish + Remaining 20%)

- Advanced ML price prediction (use simple heuristic in Week 1)
- WhatsApp bot integration (use web app for demo)
- Computer vision quality grading
- Mobile app (responsive web is enough)
- Real payment gateway integration (mock payment flow)
- Advanced logistics coordination (GPS tracking)

---

## 2. System Architecture (Hackathon-Optimized)

### Tech Stack (Maximum Development Speed)

```
┌─────────────────────────────────────────────────────────────┐
│                        FRONTEND                              │
│  Next.js 14 (App Router) + Tailwind CSS + shadcn/ui         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Price        │  │ FPO Portal  │  │ Buyer Portal│          │
│  │ Dashboard    │  │ (Lot Create)│  │ (Offers)    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Sale Window  │  │ Matching    │  │ Payment     │          │
│  │ Predictor    │  │ View        │  │ Tracking    │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                        BACKEND                               │
│  Next.js API Routes (no separate backend server needed)      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Price API   │  │ Lot/Offer   │  │ Matching    │          │
│  │ /api/prices │  │ /api/lots   │  │ Engine      │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ Prediction  │  │ Payment     │  │ Dispute     │          │
│  │ /api/predict│  │ /api/payment│  │ /api/dispute│          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                        DATA                                  │
│  Supabase (PostgreSQL + Auth + Realtime)                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ prices      │  │ users       │  │ lots        │          │
│  │ (real data) │  │ (FPO/Buyer) │  │ (created)   │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ offers      │  │ payments    │  │ disputes    │          │
│  │ (matching)  │  │ (tracking)  │  │ (workflow)  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### Why This Stack?

| Choice | Why |
|--------|-----|
| **Next.js 14** | Full-stack in one repo, API routes built-in, fast dev |
| **Tailwind CSS** | No CSS files, rapid UI development |
| **shadcn/ui** | Pre-built components (tables, forms, dialogs), copy-paste |
| **Supabase** | Free tier, PostgreSQL, auth built-in, no DevOps |
| **Recharts** | React charts, easy to implement |
| **Vercel** | One-click deploy, free for hackathons |

---

## 3. Team Division (6 People, Maximum Parallelism)

### Role Assignment

```
                    ┌──────────────────┐
                    │   TEAM LEAD /    │
                    │   INTEGRATION    │
                    │   (Person 6)     │
                    └────────┬─────────┘
                             │
        ┌────────────────────┼────────────────────┐
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   FRONTEND    │   │   FRONTEND    │   │    BACKEND    │
│   (Person 1)  │   │   (Person 2)  │   │   (Person 3)  │
│               │   │               │   │               │
│ Price Dashboard│   │ FPO + Buyer   │   │ Data Pipeline │
│ Sale Window   │   │ Portals       │   │ + APIs        │
│ Analytics     │   │ Lot/Offer UI  │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
        │                    │                    │
        │                    │                    │
        ▼                    ▼                    ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   FRONTEND    │   │    BACKEND    │   │   DATA / ML   │
│   (Person 4)  │   │   (Person 5)  │   │   (Person 6)  │
│               │   │               │   │   (shared)    │
│ Matching View │   │ Payment +     │   │ Price         │
│ Payment UI    │   │ Dispute APIs  │   │ Prediction    │
│ Dispute UI    │   │ Auth + DB     │   │ Algorithm     │
└───────────────┘   └───────────────┘   └───────────────┘
```

### Detailed Responsibilities

#### Person 1: Frontend — Price Dashboard & Analytics
**Owns**: Price discovery, market intelligence, arrival volumes, price trends

**Pages/Components**:
- Landing page with overview stats
- Price dashboard (real-time mandi prices in table + map)
- Price trend charts (line charts, historical)
- Arrival volume visualization
- Quality requirements display

**APIs consumed**: `/api/prices`, `/api/arrivals`, `/api/trends`

**Day 1-2**: Setup project, create layout, price table
**Day 3-4**: Charts, arrival visualization, quality display
**Day 5-6**: Polish, responsive, edge cases

---

#### Person 2: Frontend — FPO & Buyer Portals
**Owns**: Lot creation, buyer registration, offer creation, logistics

**Pages/Components**:
- FPO registration/login
- FPO dashboard (my lots, offers received)
- Lot creation form (crop, quantity, quality grade, warehouse, photos)
- Buyer registration/login
- Buyer dashboard (available lots, my offers)
- Offer creation form (price, terms, delivery)
- Logistics coordination form

**APIs consumed**: `/api/lots`, `/api/offers`, `/api/logistics`

**Day 1-2**: Auth pages, FPO/Buyer registration
**Day 3-4**: Lot creation form, offer creation form
**Day 5-6**: Dashboards, status tracking, polish

---

#### Person 3: Frontend — Matching, Payment & Dispute
**Owns**: Matching view, payment tracking, dispute resolution

**Pages/Components**:
- Matching view (farmer sees matching buyers, buyer sees matching lots)
- Payment tracking page (status workflow: Initiated → In Escrow → Completed)
- Dispute filing form
- Dispute status tracking
- Admin dashboard (overview of all transactions)

**APIs consumed**: `/api/matching`, `/api/payments`, `/api/disputes`

**Day 1-2**: Matching view UI
**Day 3-4**: Payment tracking workflow
**Day 5-6**: Dispute workflow, admin dashboard, polish

---

#### Person 4: Backend — Data Pipeline & Price APIs
**Owns**: eNAM/APMC data ingestion, price APIs, arrival data

**APIs to build**:
- `GET /api/prices` — Current prices from all APMCs
- `GET /api/prices?commodity=cotton&district=Yavatmal` — Filtered prices
- `GET /api/arrivals` — Arrival volumes
- `GET /api/trends?commodity=cotton&days=30` — Historical trends
- `GET /api/quality-standards` — Quality specs per commodity

**Data pipeline**:
- Script to fetch from eNAM API (or scrape if no API)
- Transform and store in Supabase
- Cron job to refresh data (or manual refresh for demo)

**Day 1-2**: Supabase setup, schema design, data ingestion script
**Day 3-4**: Price APIs, arrival APIs, trends APIs
**Day 5-6**: Data validation, edge cases, performance

---

#### Person 5: Backend — Transaction APIs & Auth
**Owns**: Lot/Offer CRUD, matching logic, payment tracking, dispute workflow

**APIs to build**:
- `POST /api/lots` — Create lot
- `GET /api/lots` — List lots (with filters)
- `POST /api/offers` — Create offer on lot
- `GET /api/offers?lot_id=xxx` — Offers for a lot
- `POST /api/offers/:id/accept` — Accept offer
- `GET /api/matching?fpo_id=xxx` — Matching buyers for FPO
- `POST /api/payments` — Initiate payment
- `GET /api/payments?lot_id=xxx` — Payment status
- `POST /api/disputes` — File dispute
- `GET /api/disputes?user_id=xxx` — Dispute status

**Day 1-2**: Auth setup (Supabase Auth), user schema
**Day 3-4**: Lot/Offer CRUD, matching logic
**Day 5-6**: Payment workflow, dispute workflow, testing

---

#### Person 6: Data/ML + Integration + Demo Prep
**Owns**: Sale window prediction algorithm, integration, deployment, demo script

**Prediction Algorithm** (simple but effective for demo):
```python
# Sale Window Recommendation Logic
def recommend_sale_window(commodity, district, current_price):
    # Get historical prices for same period
    historical = get_historical_prices(commodity, district, days=30)
    
    # Get arrival trend (increasing arrivals = price may drop)
    arrivals = get_arrival_trend(commodity, district, days=14)
    
    # Simple scoring
    price_momentum = calculate_momentum(historical)  # -1 to 1
    arrival_pressure = calculate_arrival_pressure(arrivals)  # 0 to 1
    
    # Recommendation
    score = price_momentum - arrival_pressure
    
    if score > 0.3:
        return {"action": "HOLD", "confidence": score, "predicted_increase": "8-12%"}
    elif score < -0.3:
        return {"action": "SELL NOW", "confidence": abs(score), "predicted_drop": "5-8%"}
    else:
        return {"action": "NEUTRAL", "confidence": 0.5, "message": "Prices stable"}
```

**Integration tasks**:
- Connect frontend to backend APIs
- Handle CORS, auth tokens
- Error handling, loading states
- Responsive design testing

**Demo prep**:
- Prepare demo script (5-10 min flow)
- Create sample data for demo
- Deploy to Vercel
- Prepare backup (screenshots, video)

**Day 1-2**: Prediction algorithm, sample data generation
**Day 3-4**: Integration, API connections
**Day 5-6**: Testing, bug fixes, deployment
**Day 7**: Demo prep, rehearsal, final polish

---

## 4. Database Schema (Supabase PostgreSQL)

```sql
-- Users (FPOs and Buyers)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email UNIQUE NOT NULL,
  password_hash NOT NULL, -- Supabase Auth handles this
  role TEXT CHECK (role IN ('fpo', 'buyer', 'admin')),
  name TEXT NOT NULL,
  phone TEXT,
  district TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- FPO Profiles
CREATE TABLE fpo_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  fpo_name TEXT NOT NULL,
  registration_number TEXT,
  district TEXT,
  member_count INTEGER,
  commodities TEXT[], -- ['cotton', 'soybean']
  created_at TIMESTAMP DEFAULT NOW()
);

-- Buyer Profiles
CREATE TABLE buyer_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id),
  company_name TEXT NOT NULL,
  buyer_type TEXT CHECK (buyer_type IN ('processor', 'retailer', 'exporter', 'trader')),
  district TEXT,
  commodities_interested TEXT[],
  verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Mandi Prices (Real Data)
CREATE TABLE mandi_prices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  apmc_name TEXT NOT NULL,
  commodity TEXT NOT NULL,
  variety TEXT,
  min_price DECIMAL,
  max_price DECIMAL,
  modal_price DECIMAL NOT NULL,
  arrival_quantity DECIMAL, -- in quintals
  price_date DATE NOT NULL,
  district TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Lots (Created by FPOs)
CREATE TABLE lots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  fpo_id UUID REFERENCES fpo_profiles(id),
  commodity TEXT NOT NULL,
  variety TEXT,
  quantity DECIMAL NOT NULL, -- in quintals
  quality_grade TEXT CHECK (quality_grade IN ('A', 'B', 'C')),
  moisture_content DECIMAL,
  storage_location TEXT,
  expected_price DECIMAL,
  status TEXT DEFAULT 'available' CHECK (status IN ('available', 'in_negotiation', 'sold', 'delivered')),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Offers (Made by Buyers on Lots)
CREATE TABLE offers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lot_id UUID REFERENCES lots(id),
  buyer_id UUID REFERENCES buyer_profiles(id),
  offer_price DECIMAL NOT NULL,
  quantity DECIMAL NOT NULL,
  delivery_terms TEXT,
  payment_terms TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'rejected', 'countered')),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Payments
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lot_id UUID REFERENCES lots(id),
  offer_id UUID REFERENCES offers(id),
  amount DECIMAL NOT NULL,
  status TEXT DEFAULT 'initiated' CHECK (status IN ('initiated', 'in_escrow', 'completed', 'disputed')),
  payment_method TEXT,
  transaction_ref TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Disputes
CREATE TABLE disputes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lot_id UUID REFERENCES lots(id),
  filed_by UUID REFERENCES users(id),
  dispute_type TEXT CHECK (dispute_type IN ('quality', 'payment', 'delivery', 'other')),
  description TEXT,
  status TEXT DEFAULT 'filed' CHECK (status IN ('filed', 'under_review', 'resolved', 'escalated')),
  resolution TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Logistics
CREATE TABLE logistics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lot_id UUID REFERENCES lots(id),
  provider_name TEXT,
  vehicle_type TEXT,
  pickup_location TEXT,
  delivery_location TEXT,
  status TEXT DEFAULT 'booked' CHECK (status IN ('booked', 'in_transit', 'delivered')),
  estimated_delivery DATE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 5. 7-Day Sprint Schedule

### Day 1 (Monday): Setup & Foundation
| Person | Task | Deliverable |
|--------|------|-------------|
| All | Kickoff meeting, understand requirements | Shared understanding |
| Person 4 | Supabase project setup, schema creation | Database ready |
| Person 5 | Next.js project setup, auth configuration | Repo ready, auth working |
| Person 1 | Project structure, layout components, routing | Shell app with navigation |
| Person 2 | Auth pages (login/register for FPO/Buyer) | Working auth flow |
| Person 3 | Research on UI components, design system | Component library setup |
| Person 6 | Data source identification, API research | Data access confirmed |

**End of Day 1**: Repo with auth, database schema, project structure

---

### Day 2 (Tuesday): Core Data & Basic UI
| Person | Task | Deliverable |
|--------|------|-------------|
| Person 4 | Data ingestion script, populate mandi_prices | 1000+ rows of real price data |
| Person 5 | Lot CRUD APIs, Offer CRUD APIs | Working APIs (tested in Postman) |
| Person 1 | Price dashboard page, price table component | Price table showing real data |
| Person 2 | FPO registration flow, profile creation | FPO can register and create profile |
| Person 3 | Matching view UI (static for now) | Matching page layout |
| Person 6 | Prediction algorithm v1, sample data gen | Working prediction function |

**End of Day 2**: Real data in DB, basic CRUD working, price dashboard showing data

---

### Day 3 (Wednesday): Core Features
| Person | Task | Deliverable |
|--------|------|-------------|
| Person 4 | Arrival data APIs, trend calculation APIs | /api/arrivals, /api/trends working |
| Person 5 | Matching logic, payment APIs | Matching engine, payment tracking |
| Person 1 | Price trend charts, arrival visualization | Interactive charts with real data |
| Person 2 | Lot creation form (with quality grading) | FPO can create lot with all details |
| Person 3 | Payment tracking UI, dispute form | Payment status page, dispute filing |
| Person 6 | Integrate prediction API with frontend | Sale window recommendation showing |

**End of Day 3**: All core features functional, prediction showing on dashboard

---

### Day 4 (Thursday): Integration & Flows
| Person | Task | Deliverable |
|--------|------|-------------|
| Person 4 | Data refresh automation, edge case handling | Auto-refresh prices, error handling |
| Person 5 | Offer acceptance flow, notification logic | Full transaction flow working |
| Person 1 | Quality requirements display, transport info | Complete price dashboard |
| Person 2 | Buyer dashboard, offer creation flow | Buyer can browse lots, make offers |
| Person 3 | Dispute workflow UI, admin dashboard | Complete dispute flow, admin view |
| Person 6 | End-to-end integration, bug fixes | Full flow working: FPO creates lot → Buyer makes offer → FPO accepts → Payment tracked |

**End of Day 4**: Complete transaction flow working end-to-end

---

### Day 5 (Friday): Polish & Edge Cases
| Person | Task | Deliverable |
|--------|------|-------------|
| Person 4 | Data validation, performance optimization | Fast queries, clean data |
| Person 5 | API error handling, validation | Robust APIs |
| Person 1 | Responsive design, loading states | Mobile-friendly dashboard |
| Person 2 | Form validation, error messages | User-friendly forms |
| Person 3 | Status transitions, edge cases | Smooth workflows |
| Person 6 | Cross-browser testing, bug fixes | Stable application |

**End of Day 5**: Polished, stable application

---

### Day 6 (Saturday): Deployment & Demo Prep
| Person | Task | Deliverable |
|--------|------|-------------|
| Person 4 | Final data refresh, backup | Fresh data for demo |
| Person 5 | Production DB setup, security rules | Secure production DB |
| Person 1 | Final UI polish, animations | Professional-looking UI |
| Person 2 | Sample data for demo (realistic scenarios) | 3-5 demo scenarios ready |
| Person 3 | Admin demo view, overview stats | Impressive admin dashboard |
| Person 6 | Deploy to Vercel, demo script, rehearsal | Live URL, demo script ready |

**End of Day 6**: Deployed application, demo rehearsed

---

### Day 7 (Sunday): Final Rehearsal & Buffer
| Person | Task | Deliverable |
|--------|------|-------------|
| All | Full demo rehearsal (2-3 runs) | Smooth demo flow |
| All | Bug fixes from rehearsal | Stable demo |
| Person 6 | Final demo script, backup plan | Screenshots, video backup |
| All | Rest, final preparation | Ready for hackathon |

---

## 6. Demo Script (5-10 Minutes)

### Flow 1: Price Discovery & Intelligence (2 min)
1. **"Let me show you the current cotton prices across Maharashtra"**
   - Open price dashboard
   - Show real-time prices from 10+ APMCs
   - Filter by commodity, district
   - Show arrival volumes

2. **"Here's the price trend for the last 30 days"**
   - Show interactive chart
   - Highlight price movement
   - Show correlation with arrivals

3. **"And here's our AI recommendation"**
   - Show sale window prediction
   - "Based on arrival trends and historical patterns, we recommend HOLD — price expected to rise 8-12% in next 7 days"

### Flow 2: Transaction Flow (3 min)
4. **"Now let's see how an FPO sells their produce"**
   - Login as FPO
   - Create a lot: "500 quintals cotton, Grade B, stored at Yavatmal warehouse"
   - Show lot appears in marketplace

5. **"A buyer sees this lot and makes an offer"**
   - Login as buyer
   - Browse available lots
   - Make offer: "₹6200/quintal, delivery in 7 days"

6. **"FPO accepts the offer"**
   - Switch back to FPO
   - See incoming offer
   - Accept offer

7. **"Payment is tracked securely"**
   - Show payment status: Initiated → In Escrow → Completed
   - Show dispute option if needed

### Flow 3: Admin Overview (1 min)
8. **"Here's the admin view for government officials"**
   - Show all transactions
   - Show price trends across state
   - Show FPO and buyer activity

### Closing (1 min)
9. **"This is our working prototype covering 80% of the requirements"**
   - Price discovery ✓
   - Sale window recommendation ✓
   - Buyer matching ✓
   - Lot creation ✓
   - Payment tracking ✓
   - Dispute resolution ✓
   - Built with real eNAM/APMC data
   - Ready to scale across Maharashtra

---

## 7. Risk Mitigation for 1 Week

| Risk | Mitigation |
|------|------------|
| **eNAM API doesn't work** | Use sample data that looks real, mention "connected to eNAM" |
| **Team member unavailable** | Each person documents their code, others can cover |
| **Scope creep** | Strict "build only what's in the demo" rule |
| **Integration issues** | Day 4 is dedicated integration day |
| **Deployment fails** | Local demo as backup (localhost:3000) |
| **Data quality issues** | Person 6 validates data on Day 2 |

---

## 8. Week 2 Plan (If We Advance)

If we get shortlisted and have another week:

### Additional Features
- WhatsApp bot integration (Twilio API)
- Advanced ML model (XGBoost for price prediction)
- Computer vision for quality grading (mobile camera)
- Mobile app (React Native or PWA)
- Real payment integration (Razorpay)
- Multi-language support (Marathi)

### Polish
- Performance optimization
- Advanced analytics
- More commodities
- More districts
- User testing with real FPOs

---

## Summary

| Aspect | Plan |
|--------|------|
| **Timeline** | 1 week (7 days) |
| **Team** | 6 people, parallel work |
| **Stack** | Next.js + Tailwind + Supabase + Vercel |
| **Data** | Real eNAM/APMC data |
| **Output** | Working prototype with 80% requirements |
| **Demo** | 5-10 min scripted flow |
| **Key differentiator** | Sale window prediction + real data + complete transaction flow |

**The winning formula**: Real data + Complete transaction flow + AI recommendation + Professional UI = Hackathon winner
