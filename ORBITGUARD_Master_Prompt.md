# ORBITGUARD — Master Project Prompt (Paste this into any Claude account)

## HOW TO USE THIS DOCUMENT
This is the full context + plan for our SIH hackathon project "ORBITGUARD" (PS-04: Space Debris Tracking & Satellite Collision Risk Prediction Dashboard). Paste this entire document into a new Claude conversation, then add one line at the bottom:

> "I am currently on [STEP NUMBER / PHASE NAME]. Here is what I've built so far: [paste code/status]. Tell me exactly what to do next."

Claude will then pick up context instantly without you re-explaining anything. Every teammate can work independently on different phases using this same document.

---

## 1. PROBLEM STATEMENT (PS-04)
Build a dashboard that:
- Ingests public TLE data (CelesTrak/Space-Track)
- Propagates satellite/debris orbits forward in time
- Detects conjunction (close-approach) events between object pairs
- Scores collision risk
- Visualizes orbits + flagged risk events (2D required, 3D nice-to-have)
- Produces a simple alert list of high-risk upcoming conjunctions

---

## 2. CORE THESIS / WINNING ANGLE
Most competing teams will do: TLE → SGP4 → raw distance threshold → "risk score" → pretty 3D globe. This is naive because TLEs carry real position uncertainty with no ground truth — distance alone is not risk.

**Our differentiator:** We score risk using an approximated **Probability of Collision (Pc)**, the same conceptual approach real agencies (18th Space Defense Squadron, ESA) use — not a hard distance cutoff. Then we PROVE the system works by replaying real historical collisions through it.

**One-line pitch:** *"Not just detecting collisions — predicting them with the same rigor as the agencies who actually track space."*

---

## 3. THE 5 UNIQUE PILLARS (build in this priority order)

**Pillar 1 — Pc-based risk engine (not distance thresholding)**
Approximate covariance/uncertainty growth using TLE epoch age (older TLE = more drift due to atmospheric drag in LEO). Feed into a simplified 2D Pc formulation (Foster/Alfano-style approximation). This is the technical core that sets us apart.

**Pillar 2 — Severity-weighted risk matrix**
Pull RCS (radar cross section/size class) and object type from CelesTrak SATCAT data. Risk = f(Probability, Consequence) — a tiny fragment near a dead rocket body is not equal to a fragment near an active satellite.

**Pillar 3 — Historical validation / "Time Machine" (THE key demo feature)**
Load archived TLEs from before a known real collision — target events:
- Iridium-33 / Cosmos-2251 collision (Feb 2009)
- Kosmos-1408 ASAT test debris field (Nov 2021)
Run our engine on the historical data and show it flags the real event. This is guaranteed to work on stage (unlike waiting for a live conjunction) and is our strongest "proof it's real" moment for judges.

**Pillar 4 — India-asset priority lens**
Filter SATCAT by owner code for ISRO objects (Cartosat, RISAT, EOS series, INSAT, NavIC). Build a dedicated "protect Indian space assets" view. Ties into the fact that India's own independent SSA capability (ISRO's IS4OM) is still developing — strong national-relevance narrative for judges.

**Pillar 5 — CDM-format alerts**
Format alert outputs to resemble a real Conjunction Data Message (the actual industry-standard format used by 18th SDS). Small effort, strong "we understand the domain" signal.

**Pillar 6 (stretch, only if time allows) — Cascade/Kessler simulation**
If a flagged conjunction resulted in an actual collision, use a simplified NASA Standard Breakup Model to generate a synthetic debris cloud and show which other tracked objects become newly at-risk. High wow-factor, build only after Pillars 1-5 are solid.

---

## 4. PS COMPLIANCE MAP (make sure every line below is checked off)

| PS Requirement | Our Implementation |
|---|---|
| Ingest public TLE data | Live CelesTrak API pull, cached + scheduled refresh |
| Live/near-live tracking | Scheduled re-fetch + on-demand SGP4 propagation |
| Conjunction detection via propagation | SGP4/SDP4 propagation, coarse screening then fine screening |
| Risk scoring | Pc-based probability model + severity weighting (not raw distance) |
| 2D/3D visualization | 2D orbit plot (must work) + 3D CesiumJS globe (stretch) |
| Alert list for high-risk events | CDM-style structured, ranked, filterable alert feed |

---

## 5. TECH STACK

**Backend / Orbital Mechanics**
- Python, `sgp4` library for propagation (optionally `poliastro` for validation)
- `numpy` / `scipy` for Pc probability computation
- FastAPI to serve computed data to frontend
- APScheduler or cron for periodic TLE refresh

**Data Sources**
- CelesTrak TLE + SATCAT feeds (no auth needed) — primary live data
- Space-Track.org (free account) — archived historical TLEs for the Time Machine feature

**Frontend / Visualization**
- React + Plotly or D3 for 2D orbit plots and risk matrix charts
- CesiumJS for 3D globe (stretch goal only)
- Tailwind CSS for fast dashboard styling

**Storage**
- SQLite or lightweight Postgres for object catalog + computed conjunction history

**Deployment**
- Frontend: Vercel or Netlify
- Backend: Render or Railway (free tier sufficient at this scale)

---

## 6. BUILD TIMELINE (36–48 hr hackathon window)

**Phase 1 (Hrs 0–6): Foundation**
- Pull CelesTrak TLE data; get SGP4 propagation working for 20–50 objects
- Basic 2D orbit plot rendering

**Phase 2 (Hrs 6–14): Core Detection**
- Pairwise close-approach screening over a future time window
- Baseline distance-based flagging first — get full pipeline working end-to-end

**Phase 3 (Hrs 14–24): The Differentiator**
- Implement Pc-based probability scoring using TLE-age-derived uncertainty
- Add severity weighting (RCS + object type) → full risk matrix

**Phase 4 (Hrs 24–32): Time Machine + India Lens**
- Load historical TLEs around the Feb 2009 Iridium-Cosmos event; validate detection
- Add ISRO-asset filtered dashboard view

**Phase 5 (Hrs 32–40): Polish + Alerts**
- CDM-style alert list UI
- Dashboard polish, risk matrix visualization, explanatory tooltips

**Phase 6 (Hrs 40–48): Pitch Prep**
- Rehearse demo flow (see Section 8)

**Design principle:** each phase must end with something FULLY demoable. If time runs out at hour 30, we still have a complete, impressive product — never a half-broken one.

---

## 7. JUDGING PARAMETER MAPPING (use this to shape talking points)

- **Problem Understanding** → Open pitch by explaining why distance-thresholding is naive (TLE uncertainty) — shows understanding at the physics level, not just software level.
- **Novelty & Innovation** → Pc-based scoring + historical replay + India-asset lens together = no other team will have all three.
- **Technical Feasibility** → All methods (SGP4, Foster/Alfano Pc approximation, NASA breakup model) are free, documented, and realistically buildable in the timeframe.
- **Prototype Completeness** → Layered build ensures a fully working demo at every checkpoint.
- **Impact & Societal Relevance** → Kessler syndrome risk, India's growing satellite fleet (Gaganyaan, NavIC, EO constellations), lack of accessible SSA tools for smaller institutions.
- **Scalability & Sustainability** → Architecture scales from "20 objects" to "full catalog" without redesign; could plug into a real SSA pipeline.
- **Presentation & Pitch** → Time Machine replay is the centerpiece — it cannot fail on stage, unlike waiting for a live conjunction.

---

## 8. PITCH ARC (for final presentation)

1. Hook: "Most collision tools use a distance threshold. Here's why that's dangerously naive." (~30 sec)
2. Show the Pc-based engine live on current orbital data.
3. **The moment that wins it**: "Don't take our word for it — here's our system replaying the actual 2009 Iridium-Cosmos collision using archived data. Watch it get flagged."
4. Zoom into India: dedicated view protecting Indian satellites — a capability India doesn't yet have independently.
5. Close with scalability: same architecture, more objects, real Space-Track feed, could plug into ISRO's own SSA pipeline.

---

## 9. KEY REFERENCES / PAPERS TO CITE OR STUDY

- Foster & Estes, *"A Parametric Analysis of Orbital Debris Collision Probability and Maneuver Rate for Space Vehicles,"* NASA/JSC, 1992 — Pc method foundation
- Alfano, *"A Numerical Implementation of Spherical Object Collision Probability"* — practical Pc computation
- Johnson et al., *"NASA Standard Breakup Model 2001"* — for the optional cascade feature
- Kessler & Cour-Palais (1978) — original Kessler syndrome paper
- Klinkrad, *Space Debris: Models and Risk Analysis* (book)
- Vallado, *Fundamentals of Astrodynamics and Applications* — SGP4/TLE theory
- ESA Space Debris Office annual environment reports — real-world congestion data
- CelesTrak/Kelso's articles on TLE accuracy and degradation over time

---

## 10. TEAM COORDINATION NOTE
Different teammates can own different phases/pillars in parallel:
- One person: CelesTrak ingestion + SGP4 propagation (Phase 1–2)
- One person: Pc probability engine + risk matrix (Phase 3, Pillar 1–2)
- One person: Frontend dashboard + 2D/3D visualization
- One person: Time Machine historical validation + India lens (Phase 4, Pillar 3–4)

When pasting this into your own Claude session, always state which phase/pillar you're working on and paste your current code/progress so Claude can give you precise next steps without redundant re-explanation.
