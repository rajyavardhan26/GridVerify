# ⚡ GridVerify

### Crowd-Verified Power Infrastructure Mapping

> **Hackathon MVP — 24 Hour Build**

GridVerify is a lightweight infrastructure-mapping platform that helps discover and verify **utility poles, transformers, transmission towers, and other visible electrical infrastructure** using:

* 📸 Citizen-submitted photographs
* 📍 GPS coordinates
* 🤖 Computer Vision
* 🗺️ OpenStreetMap / geospatial validation
* 📊 Explainable confidence scoring
* 🗃️ Database-backed observations
* 🌍 Interactive infrastructure map

---

# 🚀 The Idea

A large amount of transmission-level power infrastructure is already mapped.

However, distribution-level infrastructure such as:

* Utility poles
* Pole-mounted transformers
* Distribution transformers
* Small electrical assets

is still poorly mapped in many locations.

GridVerify attempts to fill this gap through **crowdsourced observations verified using AI and existing geospatial data**.

---

# 🔄 How GridVerify Works

```text
Citizen Observation
        │
        ▼
Photo + GPS + Asset Type
        │
        ▼
Computer Vision Verification
        │
        ▼
OpenStreetMap Cross-Check
        │
        ▼
Confidence Engine
        │
        ▼
Database Storage
        │
        ▼
Verification Status
        │
        ▼
Interactive Infrastructure Map
```

---

# 🎯 Problem Statement

Existing infrastructure mapping projects primarily focus on large transmission infrastructure.

The infrastructure closest to homes and neighborhoods — particularly utility poles and transformers — is much harder to maintain accurately at scale.

GridVerify addresses the question:

> **How can inexpensive citizen observations, computer vision and existing open-map information be combined to discover and confidence-score distribution-level electrical infrastructure?**

---

# 💡 Proposed Solution

A contributor takes a photograph of visible electrical infrastructure from a safe public location.

The application captures:

```text
Photo
GPS Coordinates
Asset Type
Timestamp
```

The observation is then checked using two independent verification systems.

---

# 🏗️ Supported Infrastructure Types

For the hackathon MVP:

```text
Transmission Tower / Pylon
Utility Pole
Pole-Mounted Transformer
Distribution Transformer
Substation Exterior
Unknown Power Asset
```

Do not increase the number of classes unless the core pipeline is already working.

---

# 🤖 Verification System

## 1. Computer Vision Verification

The submitted image is analyzed by a computer-vision model.

Example:

```text
User Selection:
Transmission Tower

AI Prediction:

Transmission Tower     92%
Utility Pole            5%
Other                   3%
```

The model returns:

```json
{
  "label": "transmission_tower",
  "confidence": 0.92
}
```

---

# 🗺️ 2. OpenStreetMap Verification

GridVerify searches OpenStreetMap around the submitted GPS coordinates.

Possible result:

```text
Nearby OSM Object Found

Type:
power=tower

Distance:
8.4 metres

OSM Match:
HIGH
```

Or:

```text
No matching power infrastructure found nearby.

Possible unmapped infrastructure.
```

This second result is extremely important.

It does **not automatically mean the submission is fake**.

Instead, the system may have discovered infrastructure that has not yet been mapped.

---

# 🧠 Confidence Engine

Instead of using another ML model, the hackathon MVP uses a simple explainable weighted score.

```text
Final Confidence =
(CV Confidence × 0.60)
+
(OSM Confidence × 0.40)
```

Example:

```text
CV Score  = 0.92
OSM Score = 0.85

Final Confidence

= (0.92 × 0.60) + (0.85 × 0.40)

= 0.892

≈ 89.2%
```

These weights are **prototype values**, not scientifically validated probabilities.

---

# ⚠️ When No OSM Match Exists

A missing OSM record should NOT automatically reduce a legitimate submission to rejected.

Example:

```text
AI Detection:
Pole-Mounted Transformer

CV Confidence:
94%

OSM:
No matching object found

Result:
POSSIBLE UNMAPPED ASSET
```

This is one of the main use cases of GridVerify.

---

# ✅ Verification Status

Initial MVP thresholds:

```text
Confidence >= 0.75
        ↓
VERIFIED


0.45 <= Confidence < 0.75
        ↓
PENDING


Confidence < 0.45
        ↓
REJECTED
```

These values can be adjusted after testing.

---

# 🎨 Map Status Colors

```text
🟢 Green
Verified


🟡 Yellow
Needs More Evidence / Pending


🟠 Orange
Possible Unmapped Asset / Under Review


🔴 Red
Rejected / Inconsistent
```

---

# ⭐ Core MVP Features

The hackathon demo **must demonstrate all of these**.

## Contributor

* [ ] Photo upload / capture
* [ ] Automatic GPS capture
* [ ] Asset-type selection
* [ ] Observation submission
* [ ] Verification result

## Computer Vision

* [ ] Detect infrastructure object
* [ ] Predict infrastructure class
* [ ] Return confidence score

## Geospatial Verification

* [ ] Validate latitude and longitude
* [ ] Query OpenStreetMap
* [ ] Search nearby infrastructure
* [ ] Calculate nearest-object distance
* [ ] Return match / no-match

## Verification Engine

* [ ] Combine CV score
* [ ] Combine OSM result
* [ ] Calculate final confidence
* [ ] Assign verification status

## Database

* [ ] Store submitted observations
* [ ] Store GPS
* [ ] Store AI results
* [ ] Store OSM results
* [ ] Store confidence score
* [ ] Store verification status

## Interactive Map

* [ ] Load all observations
* [ ] Display location markers
* [ ] Color markers by status
* [ ] Show observation information on click

---

# 🏛️ System Architecture

```text
┌───────────────────────────────────────────────┐
│                 React Web App                 │
│                                               │
│   Photo Upload       GPS Capture              │
│   Asset Selection    Infrastructure Map       │
└──────────────────────┬────────────────────────┘
                       │
                       │ REST API
                       ▼
┌───────────────────────────────────────────────┐
│                  FastAPI                      │
│                                               │
│   Submission API                              │
│   Verification API                            │
└───────────────┬────────────────┬──────────────┘
                │                │
                ▼                ▼
      ┌────────────────┐   ┌───────────────────┐
      │   CV Service   │   │ Geospatial Engine │
      │                │   │                   │
      │ Object         │   │ OpenStreetMap     │
      │ Detection      │   │ Overpass API      │
      └────────┬───────┘   └─────────┬─────────┘
               │                     │
               └──────────┬──────────┘
                          ▼
                ┌──────────────────────┐
                │  Confidence Engine   │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ PostgreSQL + PostGIS │
                │                      │
                │ Submissions          │
                │ Geospatial Data      │
                └──────────┬───────────┘
                           │
                           ▼
                   Interactive Map
```

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
SQLAlchemy
Pydantic
Uvicorn
```

## Database

```text
PostgreSQL
PostGIS
```

PostGIS can be used for:

```text
Distance calculations
Nearby-location queries
Spatial indexing
GeoJSON generation
```

## Machine Learning

```text
Python
PyTorch / Hosted Inference
OpenCV
Lightweight Object Detector
```

Possible hackathon-friendly approach:

```text
Roboflow Dataset
        ↓
Fine-tuned YOLO Model
        ↓
Inference
        ↓
Label + Confidence
```

A hosted inference API can also be used if training a model consumes too much hackathon time.

## Infrastructure

```text
Docker
Docker Compose
Git
GitHub
```

---

# 📁 Project Structure

```text
gridverify/
│
├── README.md
├── docker-compose.yml
│
├── backend/
│   │
│   ├── Dockerfile
│   ├── requirements.txt
│   │
│   ├── main.py
│   │
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
│   │
│   ├── requirements.txt
│   ├── labels.py
│   ├── infer.py
│   ├── train.py
│   │
│   └── model/
│
└── frontend/
    │
    ├── package.json
    ├── vite.config.ts
    ├── index.html
    │
    └── src/
        │
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

# 🗃️ Database Design

For the hackathon MVP, only one main table is necessary.

## `submissions`

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
id                  47

claimed_asset_type  transmission_tower

latitude            12.9345
longitude           77.5342

cv_label            transmission_tower
cv_confidence       0.92

osm_match           true
osm_distance_m      8.4
osm_score            0.85

final_confidence    0.89

status              VERIFIED
```

---

# 🌐 API Design

Base URL:

```text
/api
```

---

## Health Check

```http
GET /api/health
```

Response:

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

Example:

```json
{
  "submission_id": 47,
  "claimed_asset_type": "transmission_tower",
  "cv_label": "transmission_tower",
  "cv_confidence": 0.92,
  "osm_match": true,
  "osm_distance_m": 8.4,
  "final_confidence": 0.89,
  "status": "VERIFIED"
}
```

---

## Get All Submissions

```http
GET /api/submissions
```

Returns a GeoJSON `FeatureCollection`.

Example:

```json
{
  "type": "FeatureCollection",
  "features": []
}
```

The frontend uses this endpoint to display map markers.

---

# ⏱️ 24-Hour Hackathon Development Plan

The goal is **not to build everything imaginable**.

The goal is:

> Build one complete pipeline that works from photo submission to map visualization.

---

# 🕐 Hour 0–2 — Setup

### Everyone

```text
Create GitHub repository
Create branches
Create frontend/backend folders
Setup virtual environments
Setup npm project
Setup PostgreSQL
Confirm API contracts
```

Freeze the architecture after this point.

Do not redesign the entire project halfway through the hackathon.

---

# 🕒 Hour 2–6 — Build Components

## Member 1 — ML / Computer Vision

```text
Find / prepare sample dataset
      ↓
Setup inference
      ↓
Test asset recognition
      ↓
Create:

verify_image(image, claimed_type)

Returns:

{
    label,
    confidence
}
```

Target:

```text
IMAGE
  ↓
MODEL
  ↓
LABEL + CONFIDENCE
```

---

## Member 2 — Backend

Build:

```text
FastAPI application
Database connection
Submission model
POST /api/submissions
GET /api/submissions
GET /api/submissions/{id}
```

Target:

```text
Frontend
   ↓
FastAPI
   ↓
PostgreSQL
```

---

## Member 3 — Frontend

Build:

```text
React page
Photo upload
GPS capture
Asset dropdown
Submit button
Map component
```

Target UI:

```text
┌────────────────────────────┐
│       GRIDVERIFY           │
├──────────────┬─────────────┤
│              │             │
│ Submission   │    MAP      │
│ Form         │             │
│              │   ● ● ●     │
│ Upload       │             │
│ GPS          │             │
│ Asset Type   │             │
│              │             │
│ [SUBMIT]     │             │
└──────────────┴─────────────┘
```

---

## Member 4 — Geospatial / OSM

Build:

```text
GPS validation
      ↓
Overpass API query
      ↓
Nearby power object search
      ↓
Distance calculation
      ↓
OSM score
```

Function:

```python
verify_osm(latitude, longitude, asset_type)
```

Returns conceptually:

```json
{
  "match": true,
  "distance_m": 8.4,
  "score": 0.85
}
```

---

# 🕕 Hour 6–10 — First Integration

Connect:

```text
Frontend
   ↓
FastAPI
   ↓
Database
```

Then:

```text
FastAPI
   ↓
CV
```

Then:

```text
FastAPI
   ↓
OSM
```

Do not worry about beautiful UI yet.

---

# 🕙 Hour 10–14 — Complete Verification Pipeline

Implement:

```text
Photo
  ↓
CV
  ↓
CV Confidence

GPS
  ↓
OSM Check
  ↓
OSM Confidence

CV + OSM
  ↓
Confidence Engine
  ↓
Verification Status
```

The complete backend should now return:

```json
{
  "submission_id": 47,
  "cv_label": "utility_pole",
  "cv_confidence": 0.91,
  "osm_match": true,
  "osm_distance_m": 12.3,
  "final_confidence": 0.87,
  "status": "VERIFIED"
}
```

---

# 🕑 Hour 14–17 — Interactive Map

Build the GeoJSON endpoint.

```text
Database
    ↓
GeoJSON
    ↓
MapLibre
    ↓
Markers
```

Marker colors:

```text
VERIFIED     → 🟢
PENDING      → 🟡
UNMAPPED     → 🟠
REJECTED     → 🔴
```

Clicking a marker should display:

```text
Asset Type
CV Confidence
OSM Match
Distance
Final Confidence
Status
```

---

# 🕔 Hour 17–19 — End-to-End Testing

Test the complete flow.

```text
Upload Photo
      ↓
Capture GPS
      ↓
Select Asset
      ↓
Submit
      ↓
CV Detection
      ↓
OSM Verification
      ↓
Confidence Score
      ↓
Database Storage
      ↓
Marker Appears
```

Test at least:

```text
1 correct infrastructure image
1 wrong infrastructure classification
1 location with OSM match
1 location without OSM match
1 non-infrastructure image
```

---

# 🕖 Hour 19–21 — UI Polish

Only now improve presentation.

Add:

```text
Loading animation
Confidence progress bar
Status badge
Marker popup
Error messages
Responsive layout
Project logo
Basic landing text
```

Do NOT rebuild the frontend.

---

# 🕘 Hour 21–22 — Prepare Demo Data

Seed around:

```text
8–15 observations
```

Include:

```text
🟢 Verified locations
🟡 Pending locations
🟠 Possible unmapped assets
🔴 Rejected submissions
```

This makes the map visually meaningful before the live demo.

---

# 🕙 Hour 22–23 — Presentation + Demo Preparation

Prepare:

```text
Problem
Solution
Architecture
AI
Geospatial verification
Innovation
Impact
Demo
Future scope
```

Everyone should understand the complete project.

---

# 🕚 Hour 23–24 — Freeze Everything

### DO NOT ADD FEATURES.

Only:

```text
Fix bugs
Test API
Test database
Test CV
Test OSM
Test GPS
Test map
Test demo
```

Run the complete demo repeatedly.

---

# 🔥 Development Priority

## P0 — MUST WORK

```text
Map loads
     ↓
Photo + GPS + Asset Type
     ↓
Backend receives submission
     ↓
Database stores submission
     ↓
CV inference runs
     ↓
OSM verification runs
     ↓
Confidence score calculated
     ↓
Verification status assigned
     ↓
Marker appears on map
```

If this works, **you have a hackathon project**.

---

# 🟡 Stretch Features

Only attempt these if the complete P0 pipeline is working.

```text
Satellite verification

Crowd clustering

Human review dashboard

Coverage-gap heatmap

Advanced CV model

Duplicate detection

Contributor reputation

Gamification
```

Priority:

```text
P0 complete
     ↓
Coverage heatmap
     ↓
Human review
     ↓
Crowd clustering
     ↓
Satellite verification
```

---

# ❌ Do NOT Build During the Hackathon

Do not promise or attempt:

```text
Blockchain

Complex microservices

Full electrical-grid topology

Infrastructure-health prediction

Transformer voltage prediction

Capacity prediction

Failure prediction without real data

Massive production-scale architecture
```

These features make the project sound less realistic rather than more impressive.

---

# 🧪 Demo Scenario

Choose a small demo area containing:

```text
Known mapped infrastructure

Possible unmapped infrastructure

Wrong / invalid submissions
```

Seed several observations before presenting.

Then perform **one real submission live**.

---

# 🎬 Judge Demo Flow

```text
1. Open GridVerify

            ↓

2. Show existing infrastructure map

            ↓

3. Upload / capture infrastructure photo

            ↓

4. GPS automatically captured

            ↓

5. Select infrastructure type

            ↓

6. Submit observation

            ↓

7. AI identifies infrastructure

            ↓

8. OSM cross-check runs

            ↓

9. Confidence score generated

            ↓

10. Verification status shown

            ↓

11. New marker appears on map
```

Target demo time:

```text
< 2 minutes
```

---

# 🎤 Example Live Result

```text
Asset
Pole-Mounted Transformer

AI Detection
Transformer

AI Confidence
93%

OSM Verification
No Nearby Record

Final Result
Possible Unmapped Asset

Confidence
93%

Status
🟠 POSSIBLE UNMAPPED INFRASTRUCTURE
```

Then explain:

> The absence of an OSM record doesn't automatically make the observation invalid. If computer vision strongly verifies the asset but the existing map has no corresponding record, GridVerify identifies it as potentially unmapped infrastructure.

---

# 💎 Key Innovation

GridVerify does not simply ask:

```text
Where is infrastructure?
```

It asks three questions:

```text
Does this photograph actually contain
the claimed infrastructure?

              +

Does existing geospatial data already
know about it?

              +

How confident are we in the observation?
```

Therefore every observation contains **explainable evidence**.

Example:

```text
Computer Vision:
92%

OSM Match:
8.4 metres away

Final Confidence:
89%

Status:
VERIFIED
```

Instead of simply:

```text
"This tower exists."
```

---

# 🛡️ Safety

GridVerify contributors should only photograph infrastructure that is visible from legitimate public locations.

Contributors must:

```text
Stay in public areas

Never enter substations

Never cross fences

Never climb poles

Never climb towers

Never touch electrical equipment
```

Avoid capturing:

```text
Faces

Vehicle registration plates

Security arrangements

Restricted equipment

Access-control systems

Sensitive operational information
```

---

# ⚠️ What GridVerify Is NOT

GridVerify is **not**:

```text
A SCADA system

A power-grid controller

A replacement for utility GIS

A guaranteed source of ground truth

A power-flow simulator

An infrastructure-health predictor
```

Instead:

> **GridVerify is an explainable, crowdsourced infrastructure-discovery and verification platform.**

---

# 🔮 Future Scope

After the MVP proves the core idea, GridVerify could expand into:

### Crowd Verification

```text
Multiple independent submissions
            ↓
Spatial clustering
            ↓
Higher confidence
```

### Satellite Verification

```text
Citizen Observation
       +
Computer Vision
       +
OSM
       +
Satellite Imagery
       ↓
Multi-source Verification
```

### Coverage Gap Analysis

Identify regions containing:

```text
Low infrastructure mapping density
High citizen observations
Missing OSM infrastructure
```

### Human Review Dashboard

Allow trusted reviewers to inspect ambiguous observations.

### Advanced AI

Improve detection using larger infrastructure datasets and object-detection models.

---

# 📊 Success Criteria

At the end of 24 hours, the project should demonstrate:

```text
✅ Photo Upload

✅ GPS Capture

✅ Asset-Type Selection

✅ Basic Computer Vision Verification

✅ AI Confidence Score

✅ OSM / Geospatial Validation

✅ Combined Confidence Engine

✅ PostgreSQL Database Storage

✅ Verification Status

✅ Interactive Infrastructure Map

✅ End-to-End Working Demo
```

Anything beyond this is a bonus.

---

# 🏆 One-Line Pitch

> **GridVerify turns citizen photographs into explainable, confidence-scored infrastructure observations by combining computer vision, GPS and OpenStreetMap verification.**

---

# 🎤 30-Second Pitch

Large transmission infrastructure is increasingly well mapped, but the distribution infrastructure closest to neighborhoods — utility poles and transformers — remains much harder to maintain accurately.

**GridVerify targets that gap.**

A citizen safely photographs visible infrastructure. We automatically capture its location and verify the observation in two independent ways: computer vision checks whether the image actually contains the claimed asset, while geospatial validation checks whether matching infrastructure already exists nearby in OpenStreetMap.

If both agree, the observation receives high confidence.

If AI strongly identifies the infrastructure but no map record exists, GridVerify identifies it as a **possible unmapped asset**.

The result is an explainable, continuously growing infrastructure dataset rather than just another map viewer.

---

# 👥 Team Responsibilities

| Member   | Responsibility                    |
| -------- | --------------------------------- |
| Member 1 | AI / Computer Vision              |
| Member 2 | Backend + Database                |
| Member 3 | Frontend + Interactive Map        |
| Member 4 | OSM + Geospatial + Integration    |
| Member 5 | Testing + Dataset + UI support    |
| Member 6 | Integration + Demo + Presentation |

For smaller teams, combine adjacent responsibilities.

---

# 🏁 Final Rule

During the hackathon:

```text
WORKING DEMO
     >
MORE FEATURES
```

A simple pipeline that works live is far more valuable than ten advanced features that cannot be demonstrated.

---

## GridVerify

**Crowd Observation + AI + Geospatial Verification → Better Infrastructure Maps**

⚡ 📸 🤖 📍 🗺️
