# ⚡ GridVerify

### Crowd-Verified Electrical Infrastructure Mapping Using Computer Vision and Geospatial Intelligence

GridVerify is an AI-assisted crowdsourcing platform for discovering, verifying, and mapping electrical infrastructure such as **utility poles, transformers, and transmission towers**.

Citizens submit a photograph of visible electrical infrastructure from a safe public location. GridVerify captures the GPS location, analyzes the image using computer vision, cross-checks nearby infrastructure using OpenStreetMap, generates an explainable verification result, stores the observation, and displays it on an interactive map.

The core idea is simple:

```text
Photo + GPS + Asset Type
          ↓
    Computer Vision
          +
 OpenStreetMap Check
          ↓
 Verification Engine
          ↓
     Database
          ↓
   Interactive Map
```

> **GridVerify does not only consume an existing infrastructure dataset — it helps discover infrastructure that may be missing from existing maps.**

---

# 🎯 Problem Statement

Accurate information about electrical infrastructure is important for:

- Infrastructure planning
- Disaster response
- Research
- Grid modernization
- Geographic analysis
- Infrastructure accessibility
- Public-interest mapping

Large transmission infrastructure is often represented in public geospatial datasets.

However, distribution-level infrastructure such as:

- Utility poles
- Pole-mounted transformers
- Distribution transformers

can be incomplete, inconsistent, or unavailable in open datasets.

Traditional infrastructure mapping may also require trained contributors, GIS software, field surveys, or access to utility-owned datasets.

GridVerify explores a different approach:

> Can ordinary citizen observations, computer vision, GPS, and existing open geospatial data be combined to help discover and verify electrical infrastructure?

---

# 💡 Solution

GridVerify allows a user to:

1. Take or upload a photograph.
2. Capture the GPS location.
3. Select the type of infrastructure observed.
4. Submit the observation.
5. Run computer-vision verification.
6. Search nearby OpenStreetMap infrastructure.
7. Generate an explainable verification result.
8. Store the observation.
9. Display it on an interactive map.

```text
Citizen
   │
   ▼
Photo + GPS + Claimed Asset
   │
   ▼
┌─────────────────────────────┐
│      GridVerify Backend     │
└─────────────┬───────────────┘
              │
       ┌──────┴──────┐
       ▼             ▼
 Computer Vision   OSM Check
       │             │
       └──────┬──────┘
              ▼
      Verification Engine
              │
              ▼
          Database
              │
              ▼
       Interactive Map
```

---

# 🏗️ Supported Infrastructure

For the **hackathon MVP**, the computer-vision model focuses on three classes:

```text
0 → utility_pole
1 → transformer
2 → transmission_tower
```

Pole-mounted and distribution transformers are normalized into the broader `transformer` class.

Additional infrastructure classes can be introduced later, including:

- Substations
- Insulators
- Crossarms
- Electrical switches
- Power lines
- Other distribution assets

> The hackathon priority is reliable end-to-end verification, not maximizing the number of detectable classes.

---

# 🔄 Complete GridVerify Workflow

## 1. Citizen Observation

The user provides:

```text
Photo
+
GPS Coordinates
+
Claimed Asset Type
+
Timestamp
```

Example:

```text
Photo:
transformer.jpg

GPS:
12.9345, 77.5342

Claimed Asset:
Transformer
```

---

## 2. Computer Vision

The submitted photograph is analyzed by a fine-tuned YOLO object-detection model.

Example:

```text
Claimed Asset:
Transformer

        ↓

YOLO Detection:

Transformer
Confidence: 92%
```

The CV service returns a simple result:

```json
{
  "label": "transformer",
  "confidence": 0.92
}
```

The computer-vision model answers:

> **What electrical infrastructure appears in this photograph?**

It does not decide whether that infrastructure is already mapped.

That is handled separately by the geospatial verification system.

---

## 3. OpenStreetMap Cross-Check

GridVerify uses the submitted GPS coordinates to search for nearby electrical infrastructure in OpenStreetMap through the Overpass API.

Relevant OSM objects include:

```text
power=pole
power=tower
power=transformer
power=substation
```

Example:

```text
Submitted GPS
      ↓
Search nearby OSM objects
      ↓
power=transformer found
      ↓
Distance = 8.4 m
      ↓
OSM MATCH
```

Conceptual result:

```json
{
  "match": true,
  "distance_m": 8.4,
  "score": 0.85
}
```

---

# 🧠 Verification Engine

GridVerify combines visual evidence and geospatial evidence.

For observations where matching OSM infrastructure exists, an initial explainable score can be calculated as:

```text
Final Confidence =
(CV Confidence × 0.60)
+
(OSM Confidence × 0.40)
```

Example:

```text
CV Confidence  = 0.92
OSM Confidence = 0.85

Final =
(0.92 × 0.60)
+
(0.85 × 0.40)

= 0.892

≈ 89.2%
```

These weights are prototype heuristics.

They are **not intended to represent scientifically calibrated probabilities**.

---

# ⭐ Important Verification Logic

## Case 1 — CV and OSM Agree

```text
Strong CV Detection
        +
Nearby Matching OSM Asset
        ↓
     VERIFIED
```

Example:

```text
Claim:
Transformer

CV:
Transformer — 92%

OSM:
Transformer — 9 m away

Result:
VERIFIED
```

---

## Case 2 — Strong CV but No OSM Record

This is one of the most important GridVerify scenarios.

```text
Strong CV Detection
        +
Valid GPS
        +
No Nearby OSM Record
        ↓
POSSIBLE_UNMAPPED
```

Example:

```text
Claim:
Transformer

CV:
Transformer — 94%

OSM:
No transformer found nearby

Result:
POSSIBLE_UNMAPPED
```

An absent OSM object does **not** automatically mean the citizen observation is incorrect.

It may indicate that the infrastructure is missing from the existing public map.

This is central to GridVerify's purpose.

---

## Case 3 — Weak Evidence

```text
Low CV Confidence
        ↓
PENDING
```

or, when evidence strongly contradicts the submitted claim:

```text
REJECTED
```

---

# 🚦 Verification States

GridVerify uses four practical states:

### 🟢 VERIFIED

Existing evidence strongly supports the observation.

### 🟡 PENDING

Evidence exists but is not strong enough for automatic verification.

### 🟠 POSSIBLE_UNMAPPED

Strong visual evidence exists, but no corresponding nearby OSM asset was found.

### 🔴 REJECTED

Visual evidence is weak or inconsistent with the submitted claim.

---

# 🗺️ Interactive Map

Every stored observation can be displayed on the GridVerify map.

Suggested marker states:

```text
🟢 VERIFIED

🟡 PENDING

🟠 POSSIBLE_UNMAPPED

🔴 REJECTED
```

Clicking a marker can display:

```text
Asset Type
Claimed Asset
CV Prediction
CV Confidence
OSM Match
OSM Distance
Final Confidence
Verification Status
Timestamp
```

The map therefore shows not only known infrastructure but also observations that may represent **gaps in existing infrastructure data**.

---

# 🏛️ System Architecture

```text
┌──────────────────────────────────────────────┐
│                React Web App                 │
│                                              │
│  Photo Upload            GPS Capture         │
│  Asset Selection         Interactive Map     │
└────────────────────┬─────────────────────────┘
                     │
                     │ REST API
                     ▼
┌──────────────────────────────────────────────┐
│                   FastAPI                    │
│                                              │
│  Submission API          Verification API    │
└────────────┬─────────────────────┬───────────┘
             │                     │
             ▼                     ▼
┌──────────────────────┐   ┌──────────────────────┐
│      CV Service      │   │  Geospatial Service │
│                      │   │                      │
│ Fine-Tuned YOLO      │   │ OpenStreetMap        │
│ PyTorch              │   │ Overpass API         │
└──────────┬───────────┘   └──────────┬───────────┘
           │                          │
           └────────────┬─────────────┘
                        ▼
              ┌────────────────────┐
              │ Verification &     │
              │ Confidence Engine  │
              └─────────┬──────────┘
                        │
                        ▼
              ┌────────────────────┐
              │ PostgreSQL         │
              │ + PostGIS          │
              └─────────┬──────────┘
                        │
                        ▼
                 Interactive Map
```

---

# 📊 Computer Vision Dataset

For the hackathon MVP, GridVerify combines multiple public infrastructure datasets.

The different labels are normalized into:

```text
utility_pole
transformer
transmission_tower
```

---

## Dataset 1 — Pole Data

Used primarily for:

- Utility poles
- Transformers

Useful labels are mapped as:

```text
pole        → utility_pole
transformer → transformer
```

Classes not required for the MVP, such as wires, streetlights, and crossarms, can be ignored during label normalization.

---

## Dataset 2 — Utility Poles

Provides additional examples of:

- Concrete poles
- Steel poles
- Wood/timber poles
- Transformers

Normalization:

```text
concrete pole ─┐
steel pole ────┼──→ utility_pole
wood pole ─────┘

transformer ──────→ transformer
```

---

## Dataset 3 — Transmission Towers

Used primarily for:

```text
transmission tower
        ↓
transmission_tower
```

Classes such as insulators are excluded from the initial GridVerify model.

---

# 🇮🇳 Local Real-World Test Images

Public datasets may contain infrastructure from different countries, viewing angles, environments, and camera systems.

GridVerify should therefore also be tested using local photographs captured safely from public locations.

Suggested structure:

```text
custom/
├── poles/
├── transformers/
├── towers/
└── negatives/
```

Negative examples may include:

```text
Streetlights
Trees
Telecom towers
Road signs
Buildings
Empty roads
Other street infrastructure
```

These images provide an external real-world test of whether the model generalizes to Indian electrical infrastructure.

---

# 🧪 Dataset Pipeline

```text
Pole Dataset
      │
Utility Pole Dataset
      │
Transmission Tower Dataset
      │
      ▼
Label Normalization
      │
      ▼
┌─────────────────────────┐
│ 0 utility_pole          │
│ 1 transformer           │
│ 2 transmission_tower    │
└────────────┬────────────┘
             │
             ▼
        YOLO Dataset
             │
             ▼
       Model Training
             │
             ▼
          best.pt
             │
             ▼
        CV Service
```

---

# 🤖 Machine Learning Pipeline

The MVP ML workflow is:

```text
Infrastructure Images
        ↓
YOLO Annotations
        ↓
Class Normalization
        ↓
Train / Validation / Test
        ↓
Fine-Tune YOLO
        ↓
Validate
        ↓
Test on Local Photos
        ↓
best.pt
        ↓
FastAPI CV Service
```

The goal is not to train the largest possible model.

The goal is:

> **A lightweight model capable of reliably demonstrating the complete GridVerify workflow.**

---

# 🛠️ Technology Stack

## Frontend

```text
React
TypeScript
Vite
Axios
MapLibre GL JS
Browser Geolocation API
```

## Backend

```text
Python
FastAPI
Pydantic
SQLAlchemy
Uvicorn
```

## Computer Vision

```text
Python
Ultralytics YOLO
PyTorch
OpenCV
```

## Geospatial

```text
OpenStreetMap
Overpass API
PostGIS
GeoJSON
```

## Database

```text
PostgreSQL
PostGIS
```

## Development

```text
Git
GitHub
Docker
Docker Compose
```

---

# 🗄️ Database Design

For the hackathon MVP, the core database can remain deliberately simple.

## `submissions`

Suggested fields:

```text
id

claimed_asset_type

latitude
longitude

photo_path
submitted_at

cv_label
cv_confidence

osm_match
osm_distance_m
osm_score

final_confidence

status
```

Example:

```text
id                   47

claimed_asset_type   transformer

latitude             12.9345
longitude            77.5342

cv_label             transformer
cv_confidence        0.92

osm_match            true
osm_distance_m       8.4
osm_score            0.85

final_confidence     0.89

status               VERIFIED
```

PostGIS can later support:

- Nearby-location queries
- Distance calculations
- Spatial indexes
- GeoJSON generation
- Coverage analysis

---

# 🔌 API Design

The MVP API should remain small.

## Health Check

```http
GET /api/health
```

Example response:

```json
{
  "status": "ok"
}
```

---

## Create Submission

```http
POST /api/submissions
```

Input:

```text
image
latitude
longitude
asset_type
```

Conceptual response:

```json
{
  "id": 42,
  "claimed_asset": "transformer",
  "cv_label": "transformer",
  "cv_confidence": 0.91,
  "osm_match": false,
  "osm_distance_m": null,
  "final_confidence": 0.91,
  "status": "POSSIBLE_UNMAPPED",
  "latitude": 12.9345,
  "longitude": 77.5342
}
```

---

## Get Submission

```http
GET /api/submissions/{id}
```

---

## Get Map Observations

```http
GET /api/submissions
```

This endpoint can return observations as GeoJSON for direct map visualization.

---

# 📁 Proposed Repository Structure

```text
GridVerify/
│
├── README.md
├── docker-compose.yml
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   │
│   ├── routers/
│   │   ├── health.py
│   │   └── submissions.py
│   │
│   ├── services/
│   │   ├── cv_service.py
│   │   ├── osm_service.py
│   │   └── confidence_engine.py
│   │
│   └── uploads/
│
├── ml/
│   ├── model/
│   │   └── gridverify_yolo.pt
│   │
│   └── training/
│       ├── datasets/
│       ├── gridverify_dataset/
│       ├── data.yaml
│       └── train.py
│
└── frontend/
    ├── package.json
    ├── vite.config.ts
    ├── index.html
    │
    └── src/
        ├── main.tsx
        ├── App.tsx
        │
        ├── api/
        │   └── client.ts
        │
        ├── components/
        │   ├── Map.tsx
        │   ├── MarkerPopup.tsx
        │   └── SubmissionForm.tsx
        │
        ├── pages/
        │   └── Home.tsx
        │
        └── types/
            └── submission.ts
```

---

# 👥 Team Responsibilities — 4 Members

The four members work **in parallel**, not sequentially.

---

## 👤 Member 1 — AI / Computer Vision

### Responsibilities

- Inspect downloaded datasets
- Normalize dataset labels
- Merge datasets
- Prepare YOLO `data.yaml`
- Train the model
- Validate the model
- Test on real photographs
- Export `best.pt`
- Implement `cv_service.py`

Pipeline:

```text
Datasets
   ↓
Normalize
   ↓
Train YOLO
   ↓
Validate
   ↓
Real Photo Testing
   ↓
best.pt
   ↓
CV Service
```

Expected service output:

```json
{
  "label": "transformer",
  "confidence": 0.91
}
```

---

## 👤 Member 2 — Backend / Verification Engine

### Responsibilities

- Build FastAPI application
- Implement API endpoints
- Handle image uploads
- Connect CV service
- Implement confidence logic
- Combine CV and OSM results
- Generate verification status
- Handle errors

Core endpoints:

```text
GET  /api/health

POST /api/submissions

GET  /api/submissions

GET  /api/submissions/{id}
```

Member 2 should initially use **mock CV results**.

Example:

```json
{
  "label": "transformer",
  "confidence": 0.90
}
```

This allows backend development to continue while Member 1 trains the real model.

---

## 👤 Member 3 — Frontend / UI

### Responsibilities

- React application
- Photo upload/capture
- GPS capture
- Asset selector
- API communication
- Verification-result display
- Loading states
- Error states
- Responsive design
- UI polish

Target flow:

```text
Upload Photo
      ↓
Capture GPS
      ↓
Select Asset
      ↓
Submit
      ↓
Verification
      ↓
Result
```

Example result:

```text
GRIDVERIFY RESULT

Claim
Transformer

AI Detection
Transformer

AI Confidence
94%

OSM
No nearby record

Status

🟠 POSSIBLE UNMAPPED
```

---

## 👤 Member 4 — Geospatial / Database / Integration

### Responsibilities

- PostgreSQL
- PostGIS
- Database schema
- OpenStreetMap queries
- Overpass API
- Nearby-object search
- Distance calculation
- GeoJSON
- Interactive map
- Map markers
- End-to-end integration
- Debugging

Member 4 also acts as the **integration owner**.

```text
Frontend
    ↕
FastAPI
    ↕
CV + OSM
    ↕
Database
    ↕
Map
```

---

# 🤝 Parallel Development Strategy

Do **not** develop GridVerify like this:

```text
ML
 ↓
Backend
 ↓
Database
 ↓
Frontend
 ↓
Integration
```

That wastes hackathon time.

Instead:

```text
                 START
                   │
       ┌───────────┼───────────┐
       │           │           │
       ▼           ▼           ▼
    Member 1    Member 2    Member 3
      ML         Backend     Frontend
       │           │           │
       │           │           │
       └──────┐    │    ┌──────┘
              │    │    │
              ▼    ▼    ▼
                Member 4
          DB + Geo + Integration
                   │
                   ▼
               GRIDVERIFY
```

All members should agree on the API contract before development begins.

---

# ⏱️ 24-Hour Hackathon Plan

## Hour 0–1 — Setup

Everyone together:

```text
Clone repository
        ↓
Create branches
        ↓
Set up environments
        ↓
Freeze architecture
        ↓
Freeze API contract
        ↓
Freeze database schema
```

No architecture redesign after this unless absolutely necessary.

---

## Hour 1–5 — Parallel Development

### Member 1

```text
Datasets
→ Normalize
→ Merge
→ Start YOLO Training
```

### Member 2

```text
FastAPI
→ Submission API
→ Mock CV
→ Verification Logic
```

### Member 3

```text
React
→ Upload UI
→ GPS
→ Asset Selection
→ Mock Results
```

### Member 4

```text
PostgreSQL
→ PostGIS
→ OSM
→ Map
```

---

## Hour 5–8 — First Integration

```text
CV Service
      ↓
FastAPI
      ↓
Database

Frontend
      ↓
FastAPI

Database
      ↓
Map
```

Do not wait until the end of the hackathon to integrate.

---

## Hour 8–12 — End-to-End MVP

Target:

```text
PHOTO
  ↓
GPS
  ↓
ASSET TYPE
  ↓
FASTAPI
  │
  ├───────────────┐
  ▼               ▼
YOLO             OSM
  │               │
  └───────┬───────┘
          ▼
   VERIFICATION
          ↓
      DATABASE
          ↓
         MAP
```

By approximately halfway through the hackathon, this pipeline should work.

---

## Hour 12–16 — Reliability

Focus on:

```text
Bug fixing
Real photographs
OSM testing
Model testing
UI cleanup
API error handling
Database issues
Map issues
```

Reliability is more important than adding features.

---

## Hour 16–19 — Stretch Feature

Only if the MVP works.

Choose **one**:

```text
Coverage Heatmap
```

OR

```text
Human Review Dashboard
```

OR

```text
Crowd Clustering
```

Do not attempt all three.

---

## Hour 19–21 — Feature Freeze

Stop adding functionality.

Work on:

- README
- Architecture diagram
- Screenshots
- Results
- PPT
- GitHub cleanup
- Demo data

---

## Hour 21–23 — Demo Rehearsal

Run the complete demo repeatedly.

Test:

```text
What if GPS fails?

What if internet fails?

What if OSM fails?

What if YOLO detects nothing?

What if the backend restarts?

What if the database is empty?

What if the uploaded image is invalid?
```

Prepare fallback demo images and known working locations.

---

## Hour 23–24 — Final Preparation

**No new features.**

Only:

```text
Final Git Push
Final PPT
Restart Services
Test Demo
Prepare Pitch
```

---

# 🎬 Judge Demo

The ideal live demonstration should take less than two minutes.

### Step 1

Open GridVerify.

Show the map.

### Step 2

Upload a photograph of a transformer/pole/tower.

### Step 3

Allow GridVerify to capture GPS.

### Step 4

Select:

```text
Transformer
```

### Step 5

Press:

```text
VERIFY
```

### Step 6

GridVerify runs:

```text
Photo
 ↓
YOLO
 ↓
Transformer — 94%
```

and:

```text
GPS
 ↓
OpenStreetMap
 ↓
Nearby infrastructure search
```

### Step 7

Display:

```text
AI Detection
Transformer

Confidence
94%

OSM
No nearby transformer

Status

🟠 POSSIBLE_UNMAPPED
```

### Step 8

Show the new marker on the map.

Then explain:

> **GridVerify has not failed because OpenStreetMap does not contain the transformer. Discovering infrastructure that may be missing from existing maps is one of the core purposes of GridVerify.**

---

# ✨ What Makes GridVerify Different?

The novelty is **not simply YOLO**.

The novelty is **not simply OpenStreetMap**.

The novelty is **not simply crowdsourcing**.

The key idea is the complete verification workflow:

```text
Citizen Observation
        ↓
Visual Evidence
        ↓
Computer Vision
        +
Geospatial Cross-Check
        ↓
Explainable Verification
        ↓
Existing Infrastructure
        OR
Potentially Unmapped Infrastructure
        ↓
Improved Infrastructure Dataset
```

Instead of assuming that a complete infrastructure dataset already exists, GridVerify attempts to help **build and improve that dataset**.

---

# 🔁 Long-Term Data Loop

GridVerify can eventually become its own infrastructure-data generation system.

```text
Existing Public Datasets
        ↓
Initial CV Model
        ↓
GridVerify
        ↓
Citizen Observations
        ↓
CV + OSM Verification
        ↓
Human-Reviewed Observations
        ↓
GridVerify Dataset
        ↓
Model Retraining
        ↓
Better CV Model
        ↓
More Reliable Observations
```

This creates a continuously improving dataset.

---

# 🚀 Future Improvements

After the hackathon:

- Larger India-specific infrastructure dataset
- Additional infrastructure classes
- Satellite-image verification
- Multiple observations for the same asset
- Duplicate detection
- Crowd consensus
- Contributor reputation
- Human-review dashboard
- Coverage-gap heatmaps
- Advanced confidence calibration
- Active learning
- Better object detection
- Offline field-data collection
- Infrastructure-change detection

---

# ❌ Out of Scope for the Hackathon

GridVerify will **not** attempt to build:

- Blockchain infrastructure
- Complex microservices
- Full electrical-grid topology
- Power-flow simulation
- Infrastructure health prediction
- Transformer failure prediction
- Voltage prediction
- Capacity prediction
- SCADA functionality
- Production-scale utility management

The objective is a reliable working prototype.

---

# 🛡️ Safety

Electrical infrastructure can be dangerous.

GridVerify observations should only be made from legitimate and safe public locations.

Contributors should **never**:

- Enter substations
- Enter restricted utility property
- Cross fences
- Climb electrical poles
- Climb transmission towers
- Touch electrical equipment
- Interfere with infrastructure

Contributors should also avoid unnecessarily capturing:

- Faces
- Vehicle number plates
- Access-control systems
- Sensitive security information

GridVerify is a **mapping and research platform**, not an operational grid-control system.

---

# 🏁 MVP Success Criteria

The hackathon MVP is successful when a user can:

```text
Upload Photo
      +
Capture GPS
      +
Select Asset
      ↓
Computer Vision
      +
OSM Cross-Check
      ↓
Verification Decision
      ↓
Database Storage
      ↓
Interactive Map
```

Everything beyond this is optional.

> ## **WORKING DEMO > MORE FEATURES**

---

# 🎤 One-Line Pitch

> **GridVerify transforms citizen photographs into explainable, confidence-scored electrical-infrastructure observations by combining computer vision, GPS, and OpenStreetMap verification.**

---

# 🌍 Vision

Traditional infrastructure maps answer:

> **What infrastructure is already known?**

GridVerify asks:

> **What infrastructure can citizens observe, can AI verify it, does the existing map know about it, and if not, have we potentially discovered something missing?**

---

## ⚡ GridVerify

### Citizen Observation + AI + Geospatial Verification → Better Infrastructure Data
