# 🛣️ Smart Toll Plaza Management & GIS Command Center

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![Framework](https://img.shields.io/badge/Framework-Streamlit-red)
![Computer Vision](https://img.shields.io/badge/CV-FastALPR%20%7C%20YOLOv9-orange)
![GIS](https://img.shields.io/badge/GIS-Plotly%20Mapbox-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

A full-stack Python simulation of a National Highways Authority of India (NHAI)-style toll operations platform. It combines a live, camera-driven ANPR (Automatic Number Plate Recognition) pipeline, a searchable Pan-India GIS map of 2,000+ toll plazas, congestion-based dynamic pricing, and role-gated financial/security reporting — all running on a single Streamlit multi-page app.

> **Status:** Prototype / simulation. Not connected to any live FASTag, VAHAN, or NHAI production system.

---

## Table of Contents

- [System Architecture](#system-architecture)
- [Core Modules](#core-modules)
- [Role-Based Access Control](#role-based-access-control-rbac)
- [Data Layer](#data-layer)
- [Tech Stack](#tech-stack)
- [Installation & Setup](#installation--setup)
- [Demo Credentials](#demo-credentials)
- [Application Workflow](#application-workflow)
- [Dynamic Pricing Logic](#dynamic-pricing-logic)
- [Plate Validation Logic](#plate-validation-logic)
- [Security & Audit Trail](#security--audit-trail)
- [Known Limitations](#known-limitations)
- [Roadmap](#roadmap)
- [Disclaimer](#disclaimer)
- [License](#license)

---

## System Architecture

```
smart-toll-plaza-system/
├── app.py                                    # IAM login portal & role router (entry point)
├── database.py                                # CloudSQLAdapter (JSON-backed ORM) + pricing/validation logic
├── cloud_mock_db.json                         # Persistent store: transactions, audit log, shift closures, blacklist
├── TOLL_PLAZA_LIST @26 may 2026.csv            # Pan-India toll plaza geospatial registry (2,005 records)
├── toll_plaza_odisha.db                        # SQLite export of the mock DB (transactions / vahan_blacklist / security_audit)
├── India_Map_for_plaza_and_NH.pdf              # Reference map of national highway corridors
├── requirements.txt                            # Project dependencies
└── pages/
    ├── 1_📸_Live_Camera.py                     # WebRTC video feed, FastALPR inference, FASTag/cash checkout
    ├── 2_🗺️_GIS_Command_Center.py               # Pan-India plaza map, filters, node profile cards
    └── 3_📊_Analytics.py                        # Revenue, security audit & shift reconciliation dashboards
```

Streamlit's native multi-page routing is used: `app.py` is the landing/auth page, and each file under `pages/` becomes a sidebar-navigable screen, gated by the role stored in `st.session_state`.

---

## Core Modules

### 📸 Live Camera — ANPR & Payment Processing
**File:** `pages/1_📸_Live_Camera.py`

- **Continuous video pipeline** via `streamlit-webrtc`, processing every 3rd frame to balance latency and throughput.
- **Plate detection & OCR** via `fast_alpr.ALPR`, running a YOLOv9 detector (`yolo-v9-t-384-license-plate-end2end`) and a MobileViT-based OCR model (`global-plates-mobile-vit-v2-model`), cached as a Streamlit resource so the model loads once per session.
- **Confidence gating** — OCR results below a `0.50` confidence threshold are discarded.
- **Regex-validated plate formats:**
  - Standard: `AA00AA0000` (e.g. `OD02AB1234`)
  - Bharat Series: `00BH0000AA` (e.g. `21BH1234AA`)
  - Diplomatic / UN: `CD`, `CC`, `UN` prefixed short codes (e.g. `77CD12`, `UN4567`)
- **Detection queue** — a thread-safe `queue.Queue` bridges the WebRTC video processor thread and the main Streamlit thread; a `st.fragment(run_every="1s")` polls it without re-running the whole page.
- **Checkout flow:**
  - Diplomatic/VIP plates → logged as a zero-cost exempt passage.
  - Blacklisted plates (cross-checked against `vahan_blacklist`) → flagged as a security anomaly, logged to the audit trail.
  - All other plates → operator chooses **FASTag Auto-Deduct** (simulated NETC handshake with artificial latency and a ~10% random decline rate) or **Cash**.

### 🗺️ GIS Command Center — Pan-India Infrastructure Map
**File:** `pages/2_🗺️_GIS_Command_Center.py`

- Loads `TOLL_PLAZA_LIST @26 may 2026.csv` (2,005 plazas) with `@st.cache_data(ttl=3600)`, coercing latitude/longitude to numeric and dropping unmappable rows.
- Renders plazas with `plotly.express.scatter_mapbox` on a `carto-positron` basemap.
- Pre-compiles hover HTML per row with vectorized Pandas string concatenation to keep rendering fast at scale.
- Sidebar filters: **State/UT** (multiselect), **Infrastructure Type** (plaza sub-type), **Operating Authority** (concessionaire type).
- Click-to-select: clicking a node on the map (`on_select="rerun"`) surfaces a persistent detail card (plaza code, state, city, concessionaire, sub-type, coordinates) without resetting the map viewport.
- Restricted to the **Regional Director** role.

### 📊 Analytics — Financial & Security Oversight
**File:** `pages/3_📊_Analytics.py`

- **Financial Ledger tab** — total revenue, transaction count, active node, a FASTag-vs-Cash adoption donut chart, and a revenue-by-vehicle-class bar chart, backed by the `transactions` table.
- **Security Audit tab** — a live feed of `security_audit` records (blacklist interceptions, FASTag declines, VIP exemptions).
- **Shift Closure tab** *(Plaza Operator only)* — compares system-calculated expected cash against a manually entered physical cash count, logs the variance, and flags overage/shrinkage.

---

## Role-Based Access Control (RBAC)

Authentication is handled in `app.py` against a hardcoded `RBAC_USERS` dictionary (demo-only — see [Known Limitations](#known-limitations)). The resolved role and assigned node are stored in `st.session_state` and checked at the top of every page.

| Capability | Regional Director | Plaza Operator |
|---|:---:|:---:|
| Live Camera ANPR & FASTag processing | ✅ | ✅ |
| GIS Command Center (Pan-India map) | ✅ | ❌ |
| Financial Ledger & Security Audit | ✅ | ✅ |
| Shift Closure reconciliation | ❌ | ✅ |
| Node scope | All Nodes (Pan-India) | Single assigned plaza |

---

## Data Layer

`database.py` defines `CloudSQLAdapter`, a lightweight ORM-style wrapper intended to be swappable for a real Cloud SQL / Supabase / PostgreSQL backend. In its current form it persists to a single JSON file, `cloud_mock_db.json`, with four collections:

| Collection | Written by | Purpose |
|---|---|---|
| `transactions` | Live Camera checkout | Every toll transaction (plate, vehicle type, payment method, amount, exemption flag) |
| `security_audit` | Live Camera, blacklist checks | VIP exemptions, blacklist interceptions, FASTag declines |
| `shift_closures` | Analytics → Shift Closure | Operator cash reconciliation records |
| `vahan_blacklist` | Seeded at first run | Stolen/defaulter plate registry checked on every detection |

A parallel SQLite export, `toll_plaza_odisha.db`, mirrors the `transactions`, `vahan_blacklist`, and `security_audit` schemas and can be used as a starting point for migrating the adapter to a real SQL engine.

The geospatial registry (`TOLL_PLAZA_LIST @26 may 2026.csv`) contains **2,005 plazas** across **27 states/UTs**, with columns: `PLAZA CODE`, `PLAZA NAME`, `CONCESSIONAIRE TYPE`, `PLAZA SUB TYPE`, `STATE`, `CITY`, `LATITUDE`, `LONGITUDE`.

---

## Tech Stack

**Frontend / App Framework**
- Streamlit (multi-page app, `st.fragment` for polling, `st.session_state` for auth)
- Plotly Express (Mapbox scatter map, bar/pie charts)

**Computer Vision**
- `fast-alpr` (YOLOv9 plate detector + MobileViT OCR)
- `streamlit-webrtc` + `av` (live browser video stream ↔ server-side frame processing)
- OpenCV, NumPy

**Data & Backend**
- Pandas (data loading, aggregation, hover-text pre-compilation)
- Regular expressions (plate validation)
- JSON-backed `CloudSQLAdapter` (migration-ready toward Supabase/PostgreSQL)
- SQLite (reference export)

---

## Installation & Setup

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/smart-toll-plaza-system.git
cd smart-toll-plaza-system
```

### 2. Create a virtual environment

**Windows**
```bash
python -m venv venv
venv\Scripts\activate
```

**macOS / Linux**
```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

`requirements.txt` should include, at minimum:
```
streamlit
pandas
numpy
plotly
opencv-python-headless
av
streamlit-webrtc
fast-alpr
```

> **Note:** `fast-alpr` will download its YOLOv9 detector and MobileViT OCR weights on first run — this requires an internet connection the first time `1_📸_Live_Camera.py` is opened.

### 4. Place the dataset
Confirm `TOLL_PLAZA_LIST @26 may 2026.csv` sits in the project root, alongside `app.py` — the GIS module reads it by relative path.

### 5. Launch
```bash
streamlit run app.py
```

The Live Camera page additionally requires browser camera permission (WebRTC) and, for real-world use outside `localhost`, HTTPS/TURN configuration for `streamlit-webrtc`.

---

## Demo Credentials

> ⚠️ Local/demo use only. These are hardcoded plaintext values in `app.py` — **never reuse this pattern in a production system.**

| Role | Enterprise Email | Token | Node Access |
|---|---|---|---|
| Regional Director | `director@nhai.gov` | `admin88` | All Nodes (Pan-India) |
| Plaza Operator | `op1@manguli.nhai` | `toll2026` | Manguli Toll Plaza (Cuttack) |

---

## Application Workflow

```
Login (app.py)
    → RBAC validation against RBAC_USERS
    → Role stored in st.session_state
    → Sidebar page routing

Regional Director          Plaza Operator
  ├─ GIS Command Center       ├─ Live Camera
  └─ Analytics                ├─ Analytics
                               └─ Shift Closure

Live Camera pipeline:
  Camera stream (WebRTC)
    → Frame sampled (1 in 3)
    → YOLOv9 plate detection
    → MobileViT OCR
    → Confidence ≥ 0.50 & regex match
    → Pushed to detection queue
    → Polled every 1s in main thread
    → Exempt check (CD/CC/UN)
    → Blacklist check (vahan_blacklist)
    → Dynamic toll calculated
    → FASTag (simulated NETC) or Cash
    → Transaction + audit record written
```

---

## Dynamic Pricing Logic

Defined in `database.py::get_dynamic_pricing()`, based on the server's current hour:

| Window | Multiplier |
|---|---|
| 08:00 – 10:00 (Morning Peak) | × 1.15 |
| 17:00 – 20:00 (Evening Peak) | × 1.15 |
| All other hours | × 1.00 |

Base rates by vehicle class:

| Vehicle Class | Base Toll (₹) |
|---|---|
| Car/Jeep/Van | 100 |
| LCV (Light Commercial) | 160 |
| Bus/2-Axle Truck | 320 |
| Multi-Axle (3+) | 500 |
| Oversized Vehicle | 650 |

```
Final Toll = Base Toll × (1.15 if peak hour else 1.0)
```

> In the current build, `1_📸_Live_Camera.py` assigns a vehicle class deterministically from the plate string length rather than true visual classification — a placeholder pending real vehicle-classification integration (see [Roadmap](#roadmap)).

---

## Plate Validation Logic

Two independent regex layers exist across the codebase:

- `database.py` — general-purpose state-code + exemption matcher, used for exemption checks and validation helpers (`is_exempt`, `validate_indian_plate`).
- `1_📸_Live_Camera.py` — a stricter OCR-output pattern (`PLATE_REGEX`) that anchors the full string to standard, Bharat Series, and diplomatic/UN formats before a detection is accepted.

Recognized exemption prefixes: `CD`, `CC`, `UN` (diplomatic and UN corps vehicles), granted a zero-cost passage and logged to the audit trail.

---

## Security & Audit Trail

Every sensitive event is written to `security_audit` with a unique `AUD-XXXXXXXX` identifier:
- Blacklist interceptions (cross-referenced against `vahan_blacklist`)
- FASTag wallet declines
- VIP/diplomatic exemptions granted

Shift closures are logged separately with a `SHF-XXXXXXXX` identifier, recording operator, plaza, expected cash, actual cash, and variance.

Defensive data loading in the GIS module coerces coordinate columns to numeric and drops rows that fail, so a malformed CSV row cannot crash the map.

---

## Known Limitations

This is a simulation/prototype, and the following should be addressed before any production use:

- **Plaintext credentials** hardcoded in `app.py` — no hashing, no external IAM/JWT integration despite the code comment referencing Auth0/Cognito.
- **No real FASTag/NETC connectivity** — payment authorization is a `random.random()` mock with artificial latency.
- **Vehicle classification is a placeholder** — derived from plate string length, not actual visual inference.
- **JSON-file persistence** (`cloud_mock_db.json`) is not concurrency-safe and unsuitable for multi-instance deployment.
- **WebRTC in production** requires TURN/STUN and HTTPS configuration not covered by local `streamlit run`.

---

## Roadmap

- [ ] Real-time CCTV / RTSP camera integration
- [ ] Production-grade ANPR deployment (edge inference)
- [ ] Real NETC/FASTag API integration
- [ ] PostgreSQL / Supabase migration for `CloudSQLAdapter`
- [ ] Cloud deployment (containerized)
- [ ] Real-time WebSocket telemetry
- [ ] True vehicle classification (visual, not string-length heuristic)
- [ ] Multi-lane toll monitoring
- [ ] SMS/email security alerts
- [ ] Mobile operator dashboard
- [ ] Predictive traffic analytics
- [ ] AI-based anomaly detection
- [ ] Multi-factor authentication
- [ ] Encrypted audit logs

---

## Disclaimer

This project is a software simulation intended for educational, research, demonstration, and hackathon purposes. It does not represent an official NHAI system and must not be connected to real toll infrastructure, FASTag banking systems, government databases (VAHAN), or law-enforcement systems without appropriate authorization, security review, and regulatory compliance.

---

## License

Distributed under the **MIT License**.

---

<p align="center">
<strong>Smart Toll Plaza Management & GIS Command Center</strong><br/>
AI • GIS • ANPR • FASTag • Analytics • Security
</p>
