# GridVerify

### Crowd-Verified Power Infrastructure Mapping Platform

**GridVerify** is an AI-assisted geospatial platform for creating and continuously improving an open, confidence-scored dataset of visible electrical infrastructure using crowdsourced observations, computer vision, existing open-map data, and optional aerial/satellite evidence.

Instead of simply displaying existing infrastructure records, GridVerify creates an **active verification loop**:

```text
Citizen Observation
        ↓
Photo + GPS
        ↓
Computer Vision
        ↓
Geospatial Validation
        ↓
Existing Open-Map Cross-Check
        ↓
Crowd Consensus
        ↓
Optional Aerial/Satellite Evidence
        ↓
Confidence Engine
        ↓
Verified Infrastructure Dataset
        ↓
Interactive Infrastructure Map
```

---

# 1. Problem

Accurate infrastructure-location data is important for:

* disaster-response planning
* grid-resilience research
* rural electrification studies
* infrastructure planning
* outage-risk modelling
* climate-risk assessment
* academic research

However, publicly accessible power-infrastructure datasets can be incomplete or inconsistent across regions.

Existing platforms such as OpenStreetMap and infrastructure visualizers contain valuable community-generated information, but a missing asset generally remains missing until someone manually maps it.

The problem therefore is not simply:

> "How do we display electrical infrastructure?"

The problem GridVerify targets is:

> **How can we actively discover, structure, validate and continuously improve public power-infrastructure records using inexpensive citizen observations and multiple independent evidence sources?**

---

# 2. Proposed Solution

GridVerify allows a contributor to safely photograph a visible electrical-infrastructure asset from a public location.

The application captures:

```text
Photo
GPS coordinates
Timestamp
Asset type
Optional description
```

Supported MVP asset classes:

```text
Transmission tower / pylon
Utility pole
Pole-mounted transformer
Distribution transformer
Substation exterior
Unknown power asset
```

The backend then performs multiple independent checks.

### Evidence 1 — Ground Image AI

A computer-vision model determines whether the submitted photograph appears to contain the claimed infrastructure object.

Example:

```text
User claims:
Transmission Tower

CV result:
Transmission Tower → 92%
Utility Pole        → 5%
Other               → 3%
```

---

### Evidence 2 — Geospatial Validation

The system validates:

```text
GPS present?
Coordinates valid?
Photo and submitted position consistent?
Timestamp reasonable?
Duplicate submission nearby?
```

---

### Evidence 3 — Existing Open Map

GridVerify checks whether infrastructure already exists near the submitted coordinates in available open geospatial datasets.

Possible outcome:

```text
Nearby OSM power=tower found
Distance: 8.4 m

OSM consistency:
HIGH
```

Or:

```text
No matching infrastructure record found.

Possible unmapped asset.
```

The second case is especially valuable because GridVerify is designed to identify dataset gaps.

---

### Evidence 4 — Crowd Consensus

Independent observations from different users increase confidence.

Example:

```text
Submission A
Tower @ coordinate X

Submission B
Tower @ coordinate X

Submission C
Tower @ coordinate X
```

The observations are clustered geographically and combined into one candidate infrastructure asset.

---

### Evidence 5 — Aerial/Satellite Evidence

Where sufficiently high-resolution imagery is legally and technically available, an imagery patch around the submitted location can provide additional contextual evidence.

This is an **optional signal**, not the sole verification mechanism.

Small transformers and poles may not be visible in freely available medium-resolution satellite imagery.

---

# 3. Key Innovation

GridVerify is not intended to be another map viewer.

The central idea is:

> **Crowd observation → structured asset → multi-source verification → confidence score → continuously improving dataset**

Instead of answering only:

```text
Where is infrastructure already mapped?
```

GridVerify tries to answer:

```text
What infrastructure is missing?

How strong is the evidence that this asset exists?

Which records should humans verify next?

Which areas have poor mapping coverage?
```

---

# 4. Main Features

## MVP

### Contributor

* capture/upload infrastructure photo
* automatically obtain GPS coordinates
* select proposed infrastructure type
* submit observation
* receive verification result

### AI

* classify/detect infrastructure
* reject obviously unrelated images
* return model confidence
* estimate image quality

### Geospatial Engine

* validate coordinates
* find nearby submissions
* detect duplicates
* check existing open-map infrastructure
* group multiple observations into assets

### Verification Engine

* combine evidence sources
* calculate confidence score
* classify submission status

### Map

Display:

```text
Green  → Verified
Yellow → Needs more evidence
Orange → Under review
Red    → Rejected / inconsistent
```

### Dashboard

Show:

```text
Total observations
Verified assets
New unmapped candidates
Pending verification
Rejected observations
Contributors
Infrastructure by category
```

---

# 5. System Architecture

```text
┌─────────────────────────────────────┐
│            React Web App            │
│                                     │
│ Submit Photo        Infrastructure  │
│ GPS Capture         Map             │
│ Dashboard           Review Panel    │
└──────────────────┬──────────────────┘
                   │ REST API
                   ▼
┌─────────────────────────────────────┐
│             FastAPI API             │
│                                     │
│ Authentication                      │
│ Submissions                         │
│ Assets                              │
│ Statistics                          │
│ Verification                        │
└───────┬───────────┬─────────┬───────┘
        │           │         │
        ▼           ▼         ▼
┌─────────────┐ ┌────────┐ ┌──────────────┐
│ CV Service  │ │ OSM /  │ │ Verification │
│             │ │ GIS    │ │ Engine       │
│ Detector    │ │ Check  │ │              │
│ Quality     │ │        │ │ Confidence   │
└─────┬───────┘ └───┬────┘ └──────┬───────┘
      │             │              │
      └─────────────┼──────────────┘
                    ▼
          ┌──────────────────┐
          │ PostgreSQL       │
          │ + PostGIS        │
          │                  │
          │ Assets           │
          │ Observations     │
          │ Evidence         │
          │ Reviews          │
          └──────────────────┘
```

---

# 6. Technology Stack

## Frontend

```text
React
TypeScript
Vite
MapLibre GL JS
Axios
Browser Geolocation API
```

## Backend

```text
Python
FastAPI
SQLAlchemy
Pydantic
Uvicorn
```

## Database

```text
PostgreSQL
PostGIS
```

PostGIS is used for:

```text
distance searches
nearby-object queries
geospatial clustering
GeoJSON generation
bounding-box searches
```

## Machine Learning

```text
Python
PyTorch
OpenCV
Lightweight object detector/classifier
```

## Infrastructure

```text
Docker
Docker Compose
GitHub
```

---

# 7. Database Design

## users

```text
id
username
created_at
reputation_score
submission_count
```

## submissions

Every uploaded observation.

```text
id
user_id

claimed_asset_type

latitude
longitude

photo_path
captured_at
submitted_at

cv_label
cv_confidence

gps_score
osm_score
crowd_score
imagery_score

final_confidence
status
```

---

## assets

A real-world candidate infrastructure object.

Several submissions can point to the same asset.

```text
id

asset_type

latitude
longitude

confidence_score

verification_status

first_seen
last_verified

observation_count
```

---

## evidence

```text
id
submission_id

evidence_type
score
metadata
created_at
```

Possible evidence types:

```text
GROUND_CV
GPS
OSM
CROWD
AERIAL
HUMAN_REVIEW
```

---

## reviews

```text
id
asset_id
reviewer_id

decision
comment
created_at
```

---

# 8. Verification Status

Every candidate asset receives one of four states.

```text
UNVERIFIED
↓
PENDING
↓
PROBABLE
↓
VERIFIED
```

It may also become:

```text
REJECTED
```

Example:

```text
Asset #0047

Type:
Transmission Tower

Ground CV:
0.94

GPS:
0.98

Existing map match:
0.86

Crowd agreement:
0.72

Aerial evidence:
Not available

Final confidence:
0.89

Status:
VERIFIED
```

---

# 9. Confidence Engine

For the hackathon MVP, use an explainable weighted score instead of pretending to have a complex learned verification model.

Example:

```text
Confidence =
    Ground CV
  + Geographic plausibility
  + Existing-map evidence
  + Independent crowd agreement
  + Optional imagery evidence
```

Initial weights:

```text
Ground CV             40%
Geospatial checks     20%
OSM/open-data match   15%
Crowd consensus       15%
Aerial evidence       10%
```

These are prototype weights and must not be presented as scientifically validated probabilities.

If aerial imagery is unavailable, redistribute its weight instead of assigning a false negative.

For example:

```text
CV            0.91
Geo           0.96
OSM           0.00
Crowd         0.82
Aerial        unavailable
```

This does NOT necessarily mean the asset is fake.

It could mean:

```text
Potential new unmapped infrastructure
```

That is one of GridVerify's most useful outcomes.

---

# 10. Computer-Vision Pipeline

## Input

```text
Citizen photograph
        ↓
Image-quality check
        ↓
Object detection
        ↓
Infrastructure classification
        ↓
Confidence
```

MVP classes:

```text
pylon
utility_pole
transformer
substation_exterior
other
```

---

## Important Distinction

The CV system should answer:

> "Does this photograph contain the claimed object?"

It does NOT need to determine:

```text
voltage
capacity
manufacturer
operating condition
ownership
network configuration
```

Those significantly increase difficulty and are unnecessary for the hackathon.

---

# 11. Image Quality Detection

Before inference, check:

```text
resolution
blur
brightness
corruption
```

Example:

```text
Image quality: LOW

Reason:
Heavy blur

Please submit another photograph.
```

This prevents low-quality data from contaminating the dataset.

---

# 12. API Design

## Health

```http
GET /api/health
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
timestamp
```

Response:

```json
{
  "submission_id": 47,
  "status": "processing"
}
```

---

## Get Submission

```http
GET /api/submissions/{id}
```

---

## Verify Submission

```http
POST /api/submissions/{id}/verify
```

Pipeline:

```text
CV
↓
Geo checks
↓
OSM check
↓
Duplicate search
↓
Crowd evidence
↓
Confidence engine
```

---

## List Assets

```http
GET /api/assets
```

Parameters:

```text
bbox
asset_type
status
minimum_confidence
```

---

## Get Asset

```http
GET /api/assets/{id}
```

---

## GeoJSON

```http
GET /api/assets/geojson
```

Used directly by the map.

---

## Statistics

```http
GET /api/statistics
```

Response:

```json
{
  "observations": 127,
  "verified_assets": 52,
  "new_candidates": 29,
  "pending": 31,
  "rejected": 15
}
```

---

# 13. Submission Workflow

```text
1. User opens GridVerify

2. Clicks
   "Add Infrastructure"

3. Browser obtains GPS

4. User photographs visible infrastructure

5. User selects:
   Pylon / Pole / Transformer / Other

6. Submission uploaded

7. Backend validates metadata

8. CV validates image

9. Geospatial engine searches nearby assets

10. Existing open-map data checked

11. Crowd observations compared

12. Confidence calculated

13. Candidate asset created/updated

14. Map updates

15. User sees evidence breakdown
```

Target demo latency:

```text
< 5 seconds
```

for normal cached/local verification.

---

# 14. Map Interface

Main screen:

```text
┌──────────────────────────────────────────┐
│ GridVerify                    + Add Asset│
├──────────────────────────────────────────┤
│                                          │
│          🟢      🟡                      │
│                    🟢                    │
│       🟠                                 │
│                         🟢               │
│                                          │
│               MAP                        │
│                                          │
│                     🟡                   │
│                                          │
├──────────────────────────────────────────┤
│ Verified: 52 | Pending: 31 | New: 29    │
└──────────────────────────────────────────┘
```

Clicking a marker opens:

```text
Transmission Tower

Confidence: 91%

Evidence
─────────────────
Ground CV       94%
GPS             98%
OSM             No match
Crowd           89%

3 independent observations

Status:
NEW VERIFIED ASSET
```

That last result demonstrates the project's value:

> The infrastructure exists, but was absent from the reference dataset.

---

# 15. Dashboard

Show four major numbers prominently:

```text
127
Crowd observations

52
Verified assets

29
Previously unmapped candidates

31
Awaiting verification
```

Charts can include:

```text
Asset types
Verification status
Submissions over time
Coverage by region
Confidence distribution
```

Do not waste hackathon time building ten charts.

The map is the main visualization.

---

# 16. Coverage Gap Detection

A useful stretch feature is:

> "Where should somebody collect data next?"

Divide the demo area into map cells.

Calculate:

```text
Known assets
+
crowd observations
+
OSM coverage
```

Then display areas with poor evidence.

Example:

```text
Ward A
Coverage confidence: 91%

Ward B
Coverage confidence: 74%

Ward C
Coverage confidence: 23%

Recommended survey area:
Ward C
```

This turns GridVerify from just a contribution platform into an **active data-collection system**.

---

# 17. Human Verification

Do not make the mistake of claiming:

> AI = ground truth.

AI generates evidence.

Human verification remains possible.

Review screen:

```text
Candidate Asset #94

Crowd Photo
[IMAGE]

Map
[LOCATION]

AI:
Transmission tower – 93%

Existing map:
No corresponding tower

Nearby crowd observations:
2

Decision:

[ VERIFY ]
[ REQUEST MORE DATA ]
[ REJECT ]
```

---

# 18. Dataset Export

Verified/public-safe records can be exported as:

```text
GeoJSON
CSV
JSON
```

Example GeoJSON:

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [77.5946, 12.9716]
  },
  "properties": {
    "asset_type": "pylon",
    "confidence": 0.92,
    "observations": 3,
    "status": "verified"
  }
}
```

---

# 19. Repository Structure

```text
gridverify/
│
├── frontend/       React + MapLibre application
├── backend/        FastAPI REST API
├── ml/             CV training/inference
├── verification/   Multi-source verification engine
├── database/       PostgreSQL/PostGIS schema
├── data/           Demo and exported datasets
├── scripts/        Utility scripts
├── docs/           Architecture and technical docs
└── tests/          Integration/end-to-end tests
```

---

# 20. Running the Project

## Clone

```bash
git clone <repository-url>
cd gridverify
```

---

## Environment

```bash
cp .env.example .env
```

Example:

```env
DATABASE_URL=postgresql://gridverify:gridverify@localhost:5432/gridverify

BACKEND_PORT=8000
FRONTEND_PORT=5173

MODEL_PATH=ml/models/infrastructure_detector.pt

OSM_ENABLED=true
AERIAL_VERIFICATION_ENABLED=false
```

---

## Backend

```bash
python3 -m venv .venv
source .venv/bin/activate
```

For fish shell:

```fish
source .venv/bin/activate.fish
```

Then:

```bash
pip install -r requirements.txt

cd backend

uvicorn app.main:app --reload --port 8000
```

---

## Frontend

```bash
cd frontend

npm install
npm run dev
```

Frontend:

```text
http://localhost:5173
```

Backend:

```text
http://localhost:8000
```

FastAPI documentation:

```text
http://localhost:8000/docs
```

---

# 21. Docker

Start the complete development stack:

```bash
docker compose up --build
```

Services:

```text
frontend
backend
postgres
```

The ML verifier can initially run inside the backend process.

Do NOT split everything into microservices during the hackathon unless the core pipeline already works.

---

# 22. 48-Hour Development Plan

## Team Member 1 — ML/CV

Responsible for:

```text
dataset collection
data labelling
image-quality checks
infrastructure detector
inference API
evaluation
```

### Hours 0–6

Collect images for:

```text
pylon
utility pole
transformer
substation exterior
negative examples
```

### Hours 6–14

Fine-tune baseline model.

### Hours 14–20

Build:

```python
verify_image(image, claimed_type)
```

Expected result:

```json
{
  "detected": true,
  "label": "pylon",
  "confidence": 0.93
}
```

### Hours 20–30

Integrate with backend.

### Hours 30–48

Test difficult cases and prepare metrics.

---

## Team Member 2 — Backend + Database

Responsible for:

```text
FastAPI
PostgreSQL
PostGIS
submission API
asset API
verification pipeline
GeoJSON
```

First objective:

```text
POST photo
↓
save submission
↓
return submission ID
```

Then implement:

```text
nearby asset search
confidence calculation
asset creation/update
```

---

## Team Member 3 — Frontend

Responsible for:

```text
React
MapLibre
submission page
GPS
live map
evidence display
dashboard
```

First objective:

```text
Map loads
+
markers load from backend
+
Add Asset works
```

Do not start with animations.

---

## Team Member 4 — Geospatial + Integration

Responsible for:

```text
OSM integration
duplicate detection
geospatial validation
seed data
integration testing
Docker
demo
presentation
```

This person becomes the integration owner.

They must continually verify:

```text
Frontend
    ↓
Backend
    ↓
ML
    ↓
Database
    ↓
Map
```

rather than waiting until hour 40 to combine everything.

---

# 23. Development Priority

Build in this exact order.

## P0 — MUST WORK

```text
Map
↓
Submit photo
↓
Capture GPS
↓
Backend stores submission
↓
CV inference
↓
Confidence score
↓
Marker appears
```

Without this, there is no hackathon product.

---

## P1 — SHOULD WORK

```text
OSM cross-check
Duplicate detection
Evidence panel
Dashboard statistics
Human review
```

---

## P2 — NICE TO HAVE

```text
Aerial imagery verification
Coverage-gap heatmap
Contributor reputation
Gamification
GeoJSON export
Advanced CV
```

---

## P3 — DO NOT BUILD UNTIL EVERYTHING ELSE WORKS

```text
Blockchain
LLM chatbot
Native Android application
Complex microservices
Full utility-grid topology reconstruction
Transformer health prediction
Voltage prediction
AR mode
Drone support
```

They will consume time without proving the core problem.

---

# 24. Demo Scenario

Seed approximately:

```text
20 existing assets
10 pending observations
5 intentionally incorrect submissions
```

Then perform one live submission.

### Judge demo

1. Open GridVerify.
2. Show incomplete reference-map coverage.
3. Submit a real/safe demo infrastructure photograph.
4. GPS is captured.
5. CV detects the infrastructure object.
6. Existing-map data is checked.
7. Verification engine generates the confidence score.
8. New marker appears live.
9. Open marker.
10. Show every evidence source.
11. Show the new asset in dashboard statistics.

The judge should see:

```text
REAL WORLD
   ↓
AI
   ↓
VERIFICATION
   ↓
DATASET
   ↓
MAP
```

in under two minutes.

---

# 25. Evaluation Metrics

## CV

```text
Precision
Recall
F1 score
Confusion matrix
```

## Verification

Measure:

```text
Valid submissions accepted
Invalid submissions rejected
Duplicate observations detected
```

## System

Measure:

```text
Submission → result latency
API response time
Map rendering time
```

## Dataset

Measure:

```text
Total observations
Unique assets
Independent confirmations
Potentially unmapped assets identified
```

---

# 26. Example Hackathon Result

Instead of presenting:

> "We created an AI power infrastructure map."

Present measurable results:

```text
Infrastructure observations collected: 83

Unique candidate assets: 41

Multi-source verified: 29

Potential missing open-map records: 8

Invalid submissions detected: 11/12

Average verification latency: 2.4 seconds
```

Numbers used in the final presentation must come from the actual experiment.

---

# 27. Safety and Privacy

GridVerify is designed only for infrastructure visible legally from public areas or provided through authorised/open datasets.

Contributors must:

```text
stay in public areas
never enter substations
never cross fences
never climb poles/towers
never touch electrical equipment
never photograph restricted areas unlawfully
```

The application should avoid publishing sensitive operational information such as:

```text
security arrangements
access-control weaknesses
maintenance vulnerabilities
private control systems
restricted internal equipment
```

Images should avoid identifiable people and vehicle plates where possible.

For a public deployment, privacy filtering and moderation should be added before publication.

---

# 28. What GridVerify Is NOT

GridVerify is not:

```text
a utility SCADA system
a grid-control system
a vulnerability scanner
a transformer-health predictor
a replacement for official utility GIS
a guaranteed source of ground truth
```

It is:

> **a community-assisted evidence and dataset-construction platform.**

---

# 29. Future Scope

After the hackathon:

```text
active-learning pipeline

automatic OSM candidate generation

mobile application

offline field-survey mode

crowd reputation scoring

temporal infrastructure-change detection

better aerial imagery fusion

coverage-gap recommendations

utility/municipality reviewer accounts

GIS interoperability

dataset versioning

data provenance

uncertainty calibration
```

---

# 30. Research Direction

A future research question is:

> **Can heterogeneous evidence from citizen imagery, open-map records, geospatial metadata and remote imagery be fused into calibrated confidence estimates that improve infrastructure mapping in data-scarce regions?**

Possible research evaluation:

```text
Crowd only
vs
CV only
vs
OSM only
vs
Crowd + CV
vs
Crowd + CV + OSM
vs
Full evidence fusion
```

Then compare mapping precision and recall.

---

# 31. Core Principle

The system should never hide uncertainty.

Instead of:

```text
"This tower exists."
```

GridVerify should say:

```text
"We have three independent observations,
a 94% ground-image detection result,
and matching open-map evidence.

Confidence: HIGH."
```

That explainability is one of the project's most important features.

---

# 32. Hackathon Pitch

### One sentence

> **GridVerify turns citizen observations into confidence-scored public power-infrastructure data using computer vision, geospatial cross-validation and crowd consensus.**

### 30-second pitch

Reliable infrastructure data is essential for disaster planning, grid-resilience research and infrastructure analysis, but public datasets can be incomplete.

Existing maps are valuable, but they primarily show infrastructure that someone has already recorded.

GridVerify creates an active verification loop.

Citizens safely photograph visible power infrastructure from public locations. Our system validates the image using computer vision, checks its location against existing geospatial records, combines independent crowd observations and produces an explainable confidence score.

The result is not simply another infrastructure map.

It is a system for continuously **building and verifying the dataset behind the map**.
