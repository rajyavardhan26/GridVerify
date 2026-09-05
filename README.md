# GridVerify

### Crowd-Verified Power Infrastructure Mapping — Hackathon MVP

GridVerify closes the last unmapped layer of the power grid — pole-mounted
transformers and utility poles — by letting citizens photograph visible
infrastructure and cross-checking each submission against computer vision
and existing open-map records, producing a confidence-scored, continuously
verified dataset.

```text
Citizen Observation
        ↓
Photo + GPS + Asset Type
        ↓
Computer Vision Check
        ↓
Geospatial / OSM Cross-Check
        ↓
Confidence Engine
        ↓
Verified Infrastructure Dataset
        ↓
Interactive Map
```

This README covers the **agreed hackathon scope only**. Everything else
(crowd consensus, aerial imagery, dashboard, review panel, reputation,
gamification, etc.) is intentionally out of scope for the build — see
Section 9.

---

# 1. Problem

Transmission-level power grids are already reasonably well mapped —
active efforts like OpenStreetMap-based transmission mapping projects are
pushing global coverage toward near-completion in the coming years, and
countries like India already have hundreds of thousands of kilometers of
power lines mapped.

The layer below that — utility poles and pole-mounted transformers, the
infrastructure closest to actual neighborhoods — is explicitly the layer
even these well-funded efforts haven't reached yet.

The problem GridVerify targets:

> **How can we use inexpensive citizen photo observations, computer
> vision, and existing open-map data to discover and confidence-score
> the distribution-level power infrastructure that nobody has mapped yet?**

---

# 2. Proposed Solution

A contributor safely photographs a visible electrical-infrastructure
asset from a public location. The application captures:

```text
Photo
GPS coordinates
Asset type (selected by the contributor)
```

### MVP asset classes

```text
Transmission tower / pylon
Utility pole
Pole-mounted transformer
Distribution transformer
Substation exterior
Unknown power asset
```

The backend then runs two independent checks and combines them into a
single confidence score.

### Check 1 — Ground Image AI

A computer-vision model determines whether the submitted photograph
plausibly contains the claimed infrastructure object.

```text
User claims:      Transmission Tower
CV result:        Transmission Tower → 92%
                   Utility Pole        → 5%
                   Other               → 3%
```

### Check 2 — OSM / Geospatial Validation

GridVerify checks whether infrastructure already exists near the
submitted coordinates in OpenStreetMap.

```text
Nearby OSM power=tower found
Distance: 8.4 m
Match: HIGH
```

or

```text
No matching infrastructure record found.
Possible unmapped asset.
```

The second case is one of GridVerify's most valuable outcomes — it
surfaces infrastructure the official map is missing, not just infrastructure
that's already known.

---

# 3. Key Innovation

GridVerify is not another map viewer. It answers:

```text
Does this photo show what's claimed?
Does this match anything already on record?
How confident are we, and why?
```

Every score is explainable — GridVerify never just says "this exists."
It says *"92% CV match, 8.4m from an existing OSM record → HIGH confidence"*
or *"92% CV match, no matching record nearby → possible unmapped asset."*

---

# 4. MVP Features (agreed scope)

### Contributor
- Upload/capture infrastructure photo
- Automatically capture GPS coordinates
- Select proposed infrastructure type
- Submit observation
- Receive verification result

### AI
- Classify submitted photo against MVP asset classes
- Return model confidence score

### Geospatial Engine
- Validate coordinates
- Query OpenStreetMap (Overpass API) for nearby matching infrastructure
- Return match/no-match + distance

### Verification Engine
- Combine CV score + OSM score into a single confidence score
- Assign a verification status

### Map
Display all submitted assets, color-coded by status:

```text
Green  → Verified
Yellow → Needs more evidence
Orange → Under review
Red    → Rejected / inconsistent
```

---

# 5. System Architecture

```text
┌─────────────────────────────────────┐
│            React Web App            │
│                                     │
│ Submit Photo        Infrastructure  │
│ GPS Capture         Map             │
└──────────────────┬──────────────────┘
                   │ REST API
                   ▼
┌─────────────────────────────────────┐
│             FastAPI API             │
│                                     │
│ Submissions                         │
│ Verification                        │
└───────┬───────────────────┬─────────┘
        │                   │
        ▼                   ▼
┌─────────────┐     ┌──────────────────┐
│ CV Service  │     │ OSM / Geospatial │
│ Detector    │     │ Check (Overpass) │
└─────┬───────┘     └────────┬─────────┘
      │                      │
      └──────────┬───────────┘
                 ▼
       ┌──────────────────┐
       │ PostgreSQL       │
       │ + PostGIS        │
       │                  │
       │ Submissions      │
       └──────────────────┘
```

Keep the CV service running inside the backend process for the hackathon.
Do not split into microservices unless the core pipeline already works
end-to-end.

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
nearby-object queries (OSM cross-check)
GeoJSON generation for the map
```

## Machine Learning
```text
Python
PyTorch / hosted inference (e.g. Roboflow)
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

## submissions

The only table required for this scope. Every uploaded observation.

```text
id
claimed_asset_type

latitude
longitude

photo_path
submitted_at

cv_label
cv_confidence

osm_match          (boolean)
osm_distance_m
osm_score

final_confidence
status
```

No `users`, `evidence`, `assets`, or `reviews` tables are required for
this scope — those support crowd consensus, reputation, and human review,
which are out of scope (Section 9). Add them later if you have time left
over.

---

# 8. Confidence Engine

For this scope, use a simple, explainable weighted score — not a learned
model.

```text
Confidence = (CV score × 0.6) + (OSM score × 0.4)
```

These are prototype weights, not scientifically validated probabilities —
say so if a judge asks.

If there's no OSM match nearby, that does **not** mean the submission is
fake — redistribute weight to CV only and flag it explicitly as a
possible unmapped asset rather than a rejection.

### Verification status thresholds

```text
final_confidence ≥ 0.75           → VERIFIED
0.45 ≤ final_confidence < 0.75    → PENDING
final_confidence < 0.45           → REJECTED
```

(Adjust thresholds after a few real test submissions — these are starting
points, not fixed.)

---

# 9. Explicitly Out of Scope

To stay realistic for a hackathon build, the following are **not** part
of this version, even though they appear in the original full concept:

```text
Crowd consensus / independent-observation clustering
Aerial / satellite imagery evidence
Dashboard / statistics view
Human review panel
User accounts, authentication, reputation scoring
Duplicate-submission detection
Gamification
```

Do not start building any of these until every item in Section 4 works
end-to-end and is demo-ready.

---

# 10. API Design

## Health
```http
GET /api/health
```

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

## Get Submission
```http
GET /api/submissions/{id}
```

Response:
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

## Get All Submissions (for map)
```http
GET /api/submissions
```

Returns a GeoJSON FeatureCollection of all submissions for the frontend map.

---

# 11. 48-Hour Development Plan

## Team Member 1 — ML/CV
```text
0–6h    Collect/confirm sample images per asset class
        (use Roboflow Universe pole/transformer datasets)
6–14h   Wire up CV inference (hosted API or fine-tuned model)
14–20h  Build verify_image(image, claimed_type) → {label, confidence}
20–30h  Integrate with backend
30–48h  Test edge cases (blurry photos, wrong claims, non-infra photos)
```

## Team Member 2 — Backend + Database
```text
0–6h    FastAPI skeleton, PostgreSQL + PostGIS setup
6–14h   POST /submissions → save + return ID
14–24h  Overpass API integration (nearby OSM match + distance)
24–34h  Confidence engine + status assignment
34–48h  GeoJSON endpoint for map, integration testing
```

## Team Member 3 — Frontend
```text
0–8h    Map loads with MapLibre, markers from backend
8–18h   Submission page: photo upload, GPS capture, asset-type select
18–30h  Wire submission flow to backend, show verification result
30–48h  Polish map (color-coded status), test full flow live
```

## Team Member 4 — Geospatial + Integration
```text
0–6h    Confirm demo area (check OSM coverage + satellite resolution there)
6–16h   Overpass query logic, seed data for demo area
16–30h  End-to-end integration testing (frontend → backend → CV → DB → map)
30–48h  Docker Compose, final demo rehearsal, pitch prep
```

This person continuously verifies the full pipeline works, rather than
waiting until the end to combine everything.

---

# 12. Build Priority

```text
P0 — MUST WORK
Map loads
↓
Submit photo + GPS + asset type
↓
Backend stores submission
↓
CV inference runs
↓
OSM cross-check runs
↓
Confidence score calculated
↓
Marker appears on map with correct status color
```

Without this exact chain working end-to-end, there is no demo.

Everything in Section 9 stays out of scope regardless of remaining time,
unless P0 is fully working with time to spare.

---

# 13. Demo Scenario

Pick a small, pre-scouted demo area with a genuine mix of:
- Infrastructure already tagged in OSM (some submissions → match)
- Infrastructure visibly present but *not* tagged in OSM (some submissions
  → "possible unmapped asset")
- A couple of deliberately wrong submissions (e.g. a tree, a lamppost)
  to prove the CV rejects nonsense, not just accepts everything

Seed a handful of existing submissions, then perform **one live
submission** during the demo.

### Judge demo flow
```text
1. Open GridVerify — show the map with seeded points.
2. Take a real photo of a real, safe, public piece of infrastructure.
3. Submit — GPS captured, asset type selected.
4. CV detects the infrastructure object live.
5. OSM cross-check runs live (show match or "unmapped" result).
6. Confidence score + status appear.
7. New marker appears live on the map.
```

Target: under two minutes, start to finish.

---

# 14. What GridVerify Is NOT

```text
a utility SCADA system
a grid-control system
a replacement for official utility GIS
a guaranteed source of ground truth
```

It is: **a lightweight, explainable tool for discovering and
confidence-scoring the distribution-level infrastructure that isn't
mapped yet.**

---

# 15. Safety and Privacy

Contributors must:
```text
stay in public areas
never enter substations
never cross fences
never climb poles/towers
never touch electrical equipment
```

Avoid publishing sensitive operational details (security arrangements,
access-control weaknesses, restricted internal equipment) and avoid
capturing identifiable people or vehicle plates where possible.

---

# 16. Pitch

### One sentence
> GridVerify closes the last unmapped layer of the power grid — pole-mounted
> transformers and utility poles — by cross-checking citizen photos against
> existing OpenStreetMap records and computer vision, turning "probably
> exists" into a confidence-scored, verified dataset.

### 30-second pitch
Transmission grids are already well-mapped — India alone has over 458,000
km of power lines on OpenStreetMap, and active global projects are pushing
transmission coverage toward near-completion in the coming years.

But the layer below that — utility poles and pole-mounted transformers,
the infrastructure closest to actual neighborhoods — is explicitly the
layer even these efforts haven't reached yet.

GridVerify targets exactly that gap. A citizen photographs a visible pole
or transformer from a public location. We check it two independent
ways: does the image actually show what's claimed, and does it show up in
existing open-map records nearby. When it doesn't match anything on
record, that's not a failure — it's a discovery: a real asset the official
map is missing.

The result is an actively growing, explainable, confidence-scored dataset
of the exact grid layer nobody's mapped yet.
 #tree for the whole porject
 gridverify/
├── docker-compose.yml
├── README.md
│
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── main.py                    # FastAPI app entrypoint
│   ├── database.py                # PostgreSQL + PostGIS connection setup
│   ├── models.py                  # SQLAlchemy model: Submission
│   ├── schemas.py                 # Pydantic request/response schemas
│   │
│   ├── routers/
│   │   ├── health.py              # GET /api/health
│   │   └── submissions.py         # POST /submissions, GET /submissions, GET /submissions/{id}
│   │
│   ├── services/
│   │   ├── cv_service.py          # calls CV model/API, returns label + confidence
│   │   ├── osm_service.py         # Overpass API query, distance calc, match score
│   │   └── confidence_engine.py   # combines cv_score + osm_score → final_confidence + status
│   │
│   └── uploads/                   # stored submission photos (or swap for S3/local volume)
│
├── ml/
│   ├── requirements.txt
│   ├── model/                     # fine-tuned weights, if not using hosted inference
│   ├── train.py                   # optional — only if fine-tuning, not using hosted API
│   ├── infer.py                   # verify_image(image, claimed_type) → {label, confidence}
│   └── labels.py                  # MVP asset class list (pylon, pole, transformer, etc.)
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
        │   └── client.ts          # Axios instance + API calls
        │
        ├── components/
        │   ├── Map.tsx            # MapLibre map, renders markers from GeoJSON
        │   ├── MarkerPopup.tsx    # shows status, confidence, evidence on click
        │   └── SubmissionForm.tsx # photo upload, GPS capture, asset-type dropdown
        │
        ├── pages/
        │   └── Home.tsx           # combines map + submission form
        │
        └── types/
            └── submission.ts      # TS types matching backend schema
