readme file:


# 🛡️ TrustUPI — Real-Time UPI Fraud Detector

> A lightweight, rule-based fraud detection system for UPI transactions.  
> Just fast, explainable rule logic running entirely in memory.

---

## 🎯 Problem Statement

Build a system that evaluates each UPI transaction in near real-time and assigns
a fraud risk score based on rule-based logic — flagging suspicious transactions
so a fraud analyst can intervene early.

---

## ✅ Features

- Submit simulated UPI transactions via a clean web UI
- 5 rule-based fraud checks run instantly on every transaction
- Weighted risk score assigned (0–100) with reason codes
- Risk level classification: LOW / MEDIUM / HIGH / CRITICAL
- Live dashboard with total, flagged, safe counts and avg score
- Transaction history table with color-coded risk rows
- Export all flagged transactions as CSV
- Auto-refreshes every 5 seconds
- Reset everything with one click

---

## 🏗️ Architecture
┌──────────────────────────────────────┐
│          Browser (React UI)          │
│                                      │
│  • Submit Transaction Form           │
│  • Live Stats Dashboard              │
│  • Transaction Table + Risk Badges   │
│  • Export Flagged CSV                │
└──────────────┬───────────────────────┘
│ HTTP REST API (fetch)
▼
┌──────────────────────────────────────┐
│       Backend (FastAPI + Python)     │
│                                      │
│  main.py    → API routes             │
│  models.py  → Input/output schemas   │
│  scorer.py  → Runs all rules         │
│  rules.py   → 5 fraud rules          │
│  store.py   → In-memory storage      │
└──────────────┬───────────────────────┘
│
▼
┌──────────────────────────────────────┐
│       In-Memory Store (store.py)     │
│                                      │
│  transactions[]      → all records   │
│  velocity_tracker{}  → timestamps    │
│  device_tracker{}    → last device   │
│  merchant_tracker{}  → known payees  │
│  amount_history{}    → past amounts  │
└──────────────────────────────────────┘

---

## 🔍 Fraud Detection Rules

All rules live in `backend/rules.py`.
Each rule is independent, clearly named, and returns a reason code + score.
Scores are added up and capped at 100.

| # | Rule | Trigger | Reason Code | Score |
|---|------|---------|-------------|-------|
| 1 | Velocity Breach | >10 transactions in 5 minutes by same payer | `VELOCITY_BREACH` | +40 |
| 2 | Unusual Hour | Transaction between 12am – 5am | `UNUSUAL_HOUR` | +20 |
| 3 | Device Change + New Beneficiary | Device changed AND payee is new for this payer | `DEVICE_CHANGE_WITH_NEW_BENEFICIARY` | +30 |
| 4 | New Merchant High Amount | First-time payee AND amount ≥ ₹10,000 | `NEW_MERCHANT_HIGH_AMOUNT` | +30 |
| 5 | Repeated Amount Pattern | Same amount sent 3+ times by same payer | `REPEATED_AMOUNT_PATTERN` | +25 |

### Risk Level Classification

| Score | Level | Flagged? | Row Color |
|-------|-------|----------|-----------|
| 0 – 24 | LOW | No | White |
| 25 – 49 | MEDIUM | No | White |
| 50 – 74 | HIGH | Yes 🚨 | Light Red |
| 75 – 100 | CRITICAL | Yes 🚨 | Light Red |

---

## 📁 Project Structure
TrustUPI/
│
├── backend/
│   ├── main.py            # FastAPI app + all API routes
│   ├── models.py          # Pydantic input/output schemas
│   ├── rules.py           # All 5 fraud detection rules (commented)
│   ├── scorer.py          # Aggregates scores, updates memory
│   ├── store.py           # In-memory Python dicts/lists
│   └── requirements.txt   # fastapi, uvicorn only
│
├── frontend/
│   └── index.html         # Single-file React app (babel standalone)
│
└── README.md

---

## 🚀 How to Run

### Step 1 — Start the Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

- Backend: `http://localhost:8000`
- Swagger API Docs: `http://localhost:8000/docs`

### Step 2 — Open the Frontend

Open `frontend/index.html` using **Live Server** in VS Code.

- Frontend: `http://127.0.0.1:5500/frontend/index.html`

> No npm install. No build step. Opens instantly.

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/transaction` | Submit and evaluate a transaction |
| GET | `/transactions` | Get all transactions |
| GET | `/transactions/flagged` | Get only flagged transactions |
| GET | `/transactions/export-csv` | Download flagged transactions as CSV |
| GET | `/stats` | Get dashboard stats |
| DELETE | `/reset` | Clear all in-memory data |

---

## 📥 Sample Input (JSON)

```json
{
  "payer_id": "9988776655",
  "payee_id": "MERCHANT121",
  "amount": 9500,
  "timestamp": "2026-02-10T12:30:45",
  "location": "Delhi",
  "device_id": "ABC123"
}
```

---

## 🧪 Test Scenarios

| # | Scenario | Expected Score | Level | Flagged |
|---|----------|---------------|-------|---------|
| 1 | Normal daytime transaction | 0 | LOW | No |
| 2 | Transaction at 2:30am | 20 | MEDIUM | No |
| 3 | New merchant + amount ₹12,000 | 30 | MEDIUM | No |
| 4 | Midnight + new merchant + high amount | 50 | HIGH | Yes |
| 5 | Midnight + new merchant + device change + high amount | 80 | CRITICAL | Yes |
| 6 | Same payer sends 11 transactions in 5 minutes | +40 | HIGH/CRITICAL | Yes |
| 7 | Same amount sent 3+ times | +25 | varies | varies |

---

## 💡 Key Design Decisions

### Why no database?
The problem statement explicitly asked for an in-memory store for
velocity checks. Python dicts give O(1) lookup — fast enough for
real-time fraud evaluation within a session.

### Why no ML?
The problem statement said "no ML required" and preferred rule-based logic.
Rules are more explainable, auditable, and require zero training data —
critical for a fraud system where every decision must be justified to regulators.

### Why single-file frontend?
Babel standalone + one HTML file means zero build tools and zero npm install.
Keeps the focus entirely on fraud logic, not toolchain complexity.

### Why rules fire AFTER trackers update?
Scoring happens first, THEN memory is updated. This ensures a transaction
is not penalized by its own data — only by prior history.

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.9+, FastAPI, Uvicorn |
| Frontend | React 18 (CDN), Babel Standalone |
| Storage | In-memory Python dicts + lists |
| Export | Python `csv` module + StreamingResponse |
| Styling | Inline React styles |

---

## 📊 Scoring Criteria Coverage

| Criteria | Weight | How We Cover It |
|----------|--------|----------------|
| Rule Logic | 35% | 5 rule functions in rules.py, each weighted and commented |
| UI Dashboard | 25% | Live stats, transaction table, risk badges, CSV export |
| Data Handling | 20% | In-memory store, velocity tracking, pattern detection |
| Explanation & Reasoning | 10% | Reason codes returned per transaction, this README |
| Code Quality | 10% | Modular files, docstrings on every function, clean structure |

---

## 👨‍💻 Built For

**Fintech Hackathon — Problem 1: Real-Time UPI Fraud Detector (Simulated Data)**  
Duration: 2–3 hours  
Stack: Python + React  
Approach: Rule-based, explainable, lightweight
