# SIH26227: Semantic Retrieval and Multi-Temporal Change Analysis of Satellite Imagery

**Proposed Solution: Selection-Focused, MVP-Scoped Technical Plan**

---

## 1. Problem Summary

Earth-observation archives keep growing, but analysts still need to know _where and when to look_ before they can examine imagery. The challenge is to build an **on-premises, offline-capable** system that makes a multi-temporal, multi-sensor archive searchable by **meaning** (natural language or example image) and by **change over time**, while keeping conventional spatial, temporal and sensor filters and **suppressing false change**.

The solution is not positioned as "another satellite image search engine." Its central workflow is:

> **Discover → Verify → Trust**

1. **Discover** relevant locations from text, image, spatial and temporal queries.
2. **Verify** whether an apparent change is real rather than seasonal, atmospheric, registration-related or otherwise spurious.
3. **Trust** the result through evidence, calibrated confidence, date uncertainty and complete provenance.

### Six required capabilities

1. Semantic and multimodal retrieval (text-to-image, image-to-image, ranked, filterable)
2. Multi-temporal change analysis (detect, classify, date the earliest supported observation)
3. False-alarm suppression and quality handling (precision over recall)
4. Discovery and clustering of similar sites
5. Analyst workflow with provenance (review queue, confirm/reject, audit trail, export)
6. Scale, incremental ingestion and full offline operation (GeoTIFF/COG)

### Selection-facing product statement

> **From searching imagery to finding trustworthy change.**

The system does not simply report pixel or embedding differences. It combines retrieval, quality checks, spectral evidence, spatial context and, where enough observations exist, temporal evidence and persistence before promoting a candidate to a reportable event.

### Temporal dating claim

> **Temporal dating is reported at the level justified by the available observations.** A two-date case provides a bracketing interval. A multi-date case may narrow that interval only when the temporal search procedure finds sufficient, consistent evidence. The system does not claim a calendar-day change date unless the observations support that precision. Wording used throughout: "earliest supported observation found under the configured search procedure" (or "within the tested archive" when all eligible scenes were tested).

---

## 2. Core Idea

> **A semantic-search front end feeding a quality-aware change-detection back end, wrapped in an analyst review tool. Every reported event carries its evidence, provenance and explicit confidence.**

### 2.1 Precision first

Candidate detection can be deliberately loose, but **reported changes must pass strict quality and confidence requirements**. Borderline cases are not forced into yes/no decisions:

- **Report**: evidence is sufficient for the configured precision target.
- **Needs review**: potentially important, but evidence is incomplete or conflicting.
- **Reject**: evidence is insufficient or strongly supports a false alarm.

### 2.2 Honest outputs

The system does not invent an exact change date when the imagery cannot support one. For every reported event it reports an **earliest-supported-observation interval**:

- `T_last_unchanged`
- `T_first_changed`
- `interval_days`
- source scene IDs
- date quality / temporal evidence

In two-date mode the interval is the before and after acquisition dates, which is the widest and most honest bracket. It may be narrowed by a controlled temporal boundary search over clean intermediate scenes (section 7, Stage B2). That search is gated by a false-narrowing experiment, and its seasonal safeguard is a hypothesis validated before it is relied on.

Confidence is separated into three different questions that are never collapsed into one number:

- **P(real change)**
- **type evidence score / rule agreement** (not a calibrated probability)
- **date-pinning quality** (interval width, number of clean intermediate observations tested, persistence evidence where available)

### 2.3 Evidence before automation

Every event is explainable through an evidence packet containing, where available:

- evidence-model change probability
- embedding distance
- ΔNDVI, ΔMNDWI, ΔNDBI
- number of agreeing signals
- registration residual
- valid-pixel fraction and cloud/snow/shadow indicators
- view/illumination geometry (relative orbit, sun elevation)
- earliest-supported-observation interval, search mode, search coverage and temporal consistency
- spatial coherence
- persistence evidence _(multi-date mode)_
- source scenes, model and preprocessing versions

The analyst sees the evidence instead of receiving a black-box label.

### 2.4 Modular and ablatable

Each false-alarm defence is a separate stage:

- quality masking
- registration checking
- radiometric normalisation
- same-season pairing and seasonal-pair handling
- snow, shadow and view-geometry handling
- controlled temporal boundary search _(Tier 2, gated)_
- spatial context
- evidence model (calibrated logistic regression / gradient boosting)
- confidence calibration
- seasonal baseline and persistence _(multi-date mode)_

This lets the final report show which components actually reduce false alarms rather than merely listing technologies.

---

## 3. Scope: Tiers, MVP Definition and Cut Order

### Guiding principle

> **Every additional component must either improve a measured metric, reduce a documented failure mode, or satisfy an explicit requirement of the brief. Otherwise it stays out of the core system.**

### Classification used throughout this document

```text
Required by the brief
        ↓
MVP implementation (Tier 0 / Tier 1)
        ↓
Experimental extension (Tier 2, measured before being claimed)
        ↓
Stretch
```

An evaluator should not read every planned research feature as a promised capability. Tier 2 items are experiments with go/no-go gates.

### Tier 0: Evaluation Core

These must work, and nothing in Tier 1 or Tier 2 may delay them:

1. Sentinel-2 ingestion
2. Quality masking (cloud, shadow, snow)
3. Processing-baseline correction
4. Registration quality measurement (with conditional correction)
5. Semantic retrieval (text and image queries) with baselines
6. Two-date change detection with the **bracketing interval** (last unchanged / first changed)
7. False-alarm controls
8. Evidence model with calibrated confidence
9. Aggregation layer (pixel / tile / pair)
10. Reproducible evaluation with baselines and ablations
11. Offline execution

### Tier 1: Demonstrator

Required by the brief, built after Tier 0 is measured:

- analyst workbench, review queue, confirm/reject
- provenance and export
- incremental ingestion
- clustering (minimal pass, then visualisation)
- session-level feedback reranking

### Tier 2: Differentiators (experimental, gated)

Built only after Tier 0 is measured and the relevant gate passes:

- **temporal interval narrowing** (gated by the false-narrowing experiment, section 16.5)
- persistence and seasonal baseline (multi-date)
- Sentinel-1 SAR corroboration
- DEM-based terrain shadow handling
- learned change network as an extra feature
- advanced typing
- offline feedback retraining

### Tier rules

> **Tier 0 cannot be delayed by Tier 1 or Tier 2 features.**

- The **bracketing interval is Tier 0**. Only the _narrowing_ of that interval is Tier 2.
- **MVP freeze gate.** At the end of Phase 1 (after the feasibility spike and foundation), the tier assignment of every item is written down and locked. Moving an item up a tier requires removing something of similar cost.

### Scope table

| Area              | Tier 0 (core)                                                                                                                                         | Tier 1 (demonstrator)                                                                    | Tier 2 / stretch                                           |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| Sensors           | Sentinel-2 L2A                                                                                                                                        | n/a                                                                                      | Sentinel-1 SAR corroboration, Landsat, Bhuvan              |
| Retrieval         | One benchmarked embedding model, one vector index, text and image queries, metadata filters, R0–R2 baselines, prompt templates and synonym lists      | Quality boosts in ranking                                                                | Hybrid re-ranking, multi-model ensembles                   |
| Preprocessing     | Baseline correction with unit test, quality masks, measure-first registration                                                                         | n/a                                                                                      | DEM shadow/illumination handling                           |
| Change candidates | Embedding distance + ΔNDVI / ΔMNDWI / ΔNDBI, candidate-rate measurement, single-signal ablations                                                      | n/a                                                                                      | Patch-level embeddings, BSI, CVA                           |
| Evidence model    | Logistic regression (optionally shallow gradient boosting) on evidence features, including local historical variability, C0–C2 calibration comparison | n/a                                                                                      | Change network as an extra feature                         |
| Change mode       | Two-date pipeline with bracketing interval                                                                                                            | n/a                                                                                      | Controlled temporal search, persistence, seasonal baseline |
| Typing            | Rule-based with `other/unknown`, with typing evaluation                                                                                               | n/a                                                                                      | Learned typing                                             |
| Confidence        | Calibrated confidence, frozen precision-target protocol                                                                                               | n/a                                                                                      | Per-class thresholds, feedback retraining                  |
| Spatial context   | Neighbour count, components, MMU, scene-wide artifact check                                                                                           | n/a                                                                                      | Land-cover-aware thresholds                                |
| Aggregation       | Pixel / tile / pair with frozen MMU and τ                                                                                                             | n/a                                                                                      | Additional organiser formats                               |
| Discovery         | Similar-site search                                                                                                                                   | Clustering pass and visualisation                                                        | Cluster-quality analysis                                   |
| Analyst workflow  | n/a                                                                                                                                                   | Review queue, evidence view, confirm/reject, provenance, export, session-level reranking | Advanced tagging, offline retraining                       |
| Incremental index | n/a                                                                                                                                                   | Append without rebuild                                                                   | Checkpointing, recovery                                    |
| Offline           | Network-disabled restart test                                                                                                                         | Offline demo                                                                             | Reproducible deployment bundle                             |

### True MVP definition

The MVP is complete only when this chain works offline:

> **Sentinel-2 ingestion → quality + registration → tiling/indexing → semantic retrieval → two-date change detection → false-alarm suppression → earliest-supported interval → event creation → confidence/evidence → analyst review → provenance export**

The **two-date path is a complete standalone change-analysis path**, because held-out labelled evaluation cases may contain only before/after image pairs.

> **Temporal dating is reported at the level justified by the available observations.** A two-date case provides a bracketing interval. A multi-date case may narrow that interval only when the temporal search procedure finds sufficient, consistent evidence. The system does not claim a calendar-day change date unless the observations support that precision.

### Evaluation modes

**Mode A: Two-date (Tier 0)**

- Input: two observations.
- Outputs: change/no-change decision, confidence and evidence, optional type evidence score, bracketing interval.
- Metrics: precision, recall, F1, PR-AUC where appropriate.
- No persistence claim and no narrowed date.

**Mode B: Multi-date extension (Tier 2)**

- Input: a clean observation series.
- Adds controlled temporal search, seasonal baseline, persistence and break validation.
- Evaluated only where enough temporal observations exist, and only after the false-narrowing experiment.

### Cut order

**Keep at all costs**

1. ingestion
2. quality and preprocessing (including baseline correction)
3. two-date change with bracketing interval
4. false-alarm controls
5. evidence model
6. evaluation harness, baselines and ablations
7. offline execution

**Cut next if necessary**

8. clustering visualisation (keep the minimal clustering pass, because clustering is a required capability)
9. session-level feedback reranking
10. advanced typing

**Cut first**

11. learned change network
12. SAR
13. DEM handling
14. advanced multi-date modelling
15. offline retraining

> **Do not cut evaluation to save implementation time.** A smaller system with rigorous evidence is worth more than a large system with weak evaluation.

The spike (section 17) produces a time estimate for every item so this order is applied with numbers.

---

## 4. System Architecture

```text
                         OFFLINE / ON-PREMISES
                                  |
                                  v
                     GeoTIFF / COG Satellite Archive
                                  |
                                  v
              [1] PREPROCESSING + QUALITY CONTROL
              - metadata validation
              - processing-baseline check + reflectance correction
              - cloud / shadow / snow masking
              - common CRS/grid
              - registration: measure -> correct if needed -> measure
              - tiling + quality records
              - accept / flag
                                  |
               +------------------+------------------+
               |                                     |
               v                                     v
      [2] SEMANTIC DISCOVERY                 [3] TWO-DATE CHANGE ENGINE
      - text -> image                         (Tier 0)
      - image -> image                        - candidate generation
      - metadata filters                      - spectral / embedding /
      - similar sites, clustering               quality / spatial evidence
               |                              - evidence model
               |                              - calibrated confidence
               |                                     |
               +------------------+------------------+
                                  |
                                  v
                       [4] AGGREGATION + EVENT LAYER
                       - event merging, footprints
                       - pixel / tile / event / pair outputs
                       - evidence packet, provenance
                                  |
                       +----------+----------+
                       |                     |
                    CHANGE               NO CHANGE
                       |
                       v
                [5] TEMPORAL EVIDENCE
                - bracketing interval (Tier 0)
                - controlled temporal search (Tier 2, gated)
                       |
                +------+------+
                |             |
           consistent      ambiguous
                |             |
         narrow interval   flag + bracket
                       |
                       v
                [6] ANALYST WORKBENCH (Tier 1)
                - map, before/after, evidence
                - confirm/reject, reranking
                - provenance export
                       |
                       v
                [7] INCREMENTAL INDEX (Tier 1)
                New scene -> process -> append -> searchable
```

### Design principle

The architecture separates:

- **Discovery**: "Where should I look?"
- **Verification**: "Did something really change?"
- **Trust**: "Why should I believe this result?"

It also separates three claims the system can make: what it can **detect**, what it can **verify**, and what it can **date**.

---

## 5. Component Design

### 5.1 Geospatial Ingestion and Preprocessing

- **Input:** Sentinel-2 L2A (surface reflectance) as GeoTIFF/COG, with product metadata.
- **Quality masking:** combine SCL and cloud-probability information where available. The usable mask is computed **per observation pair**: a pixel is usable only when it satisfies the quality policy on both dates. The mask covers cloud, cloud shadow, snow/ice and haze (section 6.5).
- **Processing-baseline check.** Record the product's processing baseline in the metadata. Sentinel-2 processing baseline 04.00 (early 2022) is reported to have introduced a reflectance offset. Scenes are converted to a common reflectance convention on ingest using the offset fields in the product metadata, and the preprocessing version records that this was done. **Verify the exact offset handling against ESA's product documentation during the spike.**
- **Per-satellite differences.** Store which satellite (S2A, S2B, S2C) produced each scene. Small radiometric differences between units are handled by normalisation and reported as a quality feature.
- **Grid:** one documented CRS and pixel grid for the archive or AOI, with a documented resampling method. MNDWI and NDBI use the 20 m SWIR band, so the resampling to the 10 m grid is documented.
- **Registration: measure, correct only when needed, measure again.**

  ```text
  native georeferencing
          ↓
  measure registration residual (phase correlation / AROSICS-style)
          ↓
  acceptable? ──yes──> keep native alignment (no resampling)
          │
          no
          ↓
  co-register the later scene
          ↓
  measure residual again
          ↓
  still above threshold? ──yes──> flag pair, down-weight or exclude
  ```

  Resampling every scene unconditionally can itself introduce edge differences, so correction is applied only when the measured residual exceeds a threshold declared before evaluation. The residual before and after is stored in the evidence packet. The effect is measured in the ablation (section 16.8).

- **Normalisation:** IR-MAD or stable-pixel regression per tile pair, using only valid pixels. It is not applied blindly if it risks suppressing real change.
- **Tiling:** fixed chips, benchmarked at 128×128, 256×256 and 512×512 before locking the production size. Every tile records:
  - tile ID, source-scene ID, acquisition time, geo-bounds
  - sensor, satellite unit, band/composite information
  - relative orbit, sun elevation/azimuth (view and illumination geometry)
  - valid fraction, cloud/snow/shadow fraction
  - processing baseline and preprocessing version
  - registration residual before and after correction
- **Storage:** arrays or per-tile COGs; metadata in PostgreSQL + PostGIS or SQLite + SpatiaLite.
- **Incremental ingestion:** a new scene is validated, preprocessed, tiled, embedded and appended to the vector index with a metadata record. Existing tiles are not reprocessed unnecessarily.
- **Quality record:** every tile/observation retains enough information to decide whether it is safe to compare.

#### Ingestion validation block

Every ingested scene runs this sequence, and its outcome is stored:

```text
Scene
  ↓
metadata validation
  ↓
processing-baseline check
  ↓
reflectance normalisation
  ↓
quality-mask validation
  ↓
registration residual measurement
  ↓
accept / flag
```

#### Processing-baseline unit test

Do not merely state that the correction exists. Take a known no-change area (stable bare ground, urban core or deep water) observed on both sides of a processing-baseline change and compare:

```text
raw index difference        vs        corrected index difference
```

**Pass condition:** the artificial contribution of the baseline change to ΔNDVI / ΔMNDWI / ΔNDBI over the stable area is reduced after correction (a threshold on the reduction is declared before the test). The result goes in the evaluation report. If it does not pass, the correction is fixed or the affected pairs are flagged and excluded.

### 5.2 Semantic Retrieval

The retrieval model is an **engineering choice to benchmark**, not a fixed selling point.

- **Candidate model class:** pretrained remote-sensing vision-language / image-text embedding model. Shortlist several (for example GeoRSCLIP, SkyCLIP, RemoteCLIP, SenCLIP-style and multispectral CLIP variants) and do not default to one. Published comparisons on Sentinel-2 retrieval rank these models differently, so the team's own benchmark decides.
- **Input compatibility:** many remote-sensing CLIP models take RGB only. Verify whether the selected model supports multispectral Sentinel-2 input or needs RGB/false-colour rendering.
- **Offline packaging:** weights, tokenizer/preprocessing code and licence information are packaged before the final model is selected. Weight availability and licences are checked per candidate.

#### Mandatory retrieval baselines

| ID  | Configuration                                        | Notes                                                                                                                                                |
| --- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| R0  | Simple baseline                                      | Image queries: similarity on mean spectral/index features or histograms. Text queries: a general-purpose (non-remote-sensing) CLIP as the reference. |
| R1  | Best candidate embedding model                       | Chosen on the team's own retrieval set                                                                                                               |
| R2  | R1 + prompt-template averaging and synonym expansion | Tests the robustness strategy                                                                                                                        |

Report for each: Recall@K, Precision@K, mAP where appropriate, latency, and storage/index size.

#### Query robustness test

Evaluate each configuration on four query groups:

```text
seen wording        (queries used to tune templates)
paraphrased wording (same concept, different sentence)
synonym wording     (same concept, different key terms)
unseen concept      (concept not used in tuning)
```

The prompt templates, synonym lists and expansion rules are rule-based and offline, with no external language model or API. A gap between seen and unseen groups is reported, not hidden.

**Queries supported**

1. Text → image
2. Image → image
3. Text/image + spatial filter
4. Text/image + temporal filter
5. Text/image + sensor/quality filter
6. Similar-site discovery from an analyst-selected event

**Retrieval output** shows similarity score, location, acquisition date, source scene, quality flags, thumbnail and optional change/event status.

FAISS handles vector similarity. **Metadata filtering remains a database concern**; FAISS is not treated as the metadata database. For large-scale filtering, the implementation may over-retrieve vector candidates, then apply spatial/temporal/quality filtering and reranking. A database with vector support (for example PostGIS with a vector extension) is an alternative worth benchmarking because it filters and searches in one place.

**Resolution honesty.** At Sentinel-2's 10 m scale, individual vehicles and many small objects are not reliably visible. Semantic queries are scoped to patterns that are meaningfully represented, such as settlements, built-up areas, water bodies, cleared land, agricultural patterns, road corridors and large structural changes. If higher-resolution imagery is supplied by the organisers, the same architecture can ingest it with an appropriate model.

---

## 6. False-Alarm Controls

False-alarm suppression is a **first-class subsystem**, not a final cosmetic filter.

### 6.1 Evidence sources

```text
                 Candidate Difference
                         |
       +-----------------+------------------+
       |                 |                  |
   Image/Embedding   Spectral Change    Data Quality
     Evidence        NDVI/MNDWI/NDBI     Cloud/Shadow/Snow
       |                 |               Registration
       |                 |               Valid pixels
       +-----------------+------------------+
                         |
                  Season handling
        same-season pairing (two-date, MVP)
        seasonal baseline + persistence
        (multi-date extension)
                         |
                  Spatial Evidence
              coherence + scene-wide QC
                         |
                         v
                  Evidence Fusion
                         |
          +--------------+--------------+
          |              |              |
        REPORT      NEEDS REVIEW       REJECT
```

### 6.2 False-alarm taxonomy

Every false-alarm experiment reports performance **by category**, not as one overall number (section 16.3).

| Code | Class                                                                 | Main defences                                                                      |
| ---- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| F1   | Seasonal vegetation                                                   | Same-season pairing, seasonal pairs as negatives, soft cropland handling           |
| F2   | Cloud / haze                                                          | SCL and cloud-probability masks, scene-wide artifact check                         |
| F3   | Snow and ice                                                          | Snow/ice mask, snow-state pairing                                                  |
| F4   | Terrain shadow, cloud shadow, illumination and view-angle differences | Shadow masks, same-season and same-relative-orbit pairing, sun-elevation feature   |
| F5   | Registration error                                                    | Measure-first conditional co-registration, residual as evidence                    |
| F6   | Processing baseline / sensor or radiometric inconsistency             | Baseline correction with unit test, per-satellite feature, normalisation           |
| F7   | Water fluctuation                                                     | Seasonal-pair negatives, MNDWI context, historical water behaviour where available |
| F8   | Agricultural cycle                                                    | Same-season pairing, soft cropland handling, land-cover context (stretch)          |
| F9   | Mixed / unknown                                                       | Needs-review band                                                                  |

Multi-date-only classes (Tier 2): transient disturbance, insufficient observations, genuine persistent change.

The system should demonstrate that it can distinguish at least some of these classes rather than simply showing before/after differences.

### 6.3 Scene-wide artifact detection

The **scene-wide candidate-rate check** is a key safety mechanism. If an unusually large fraction of a scene becomes a candidate, widespread neighbour agreement is not treated as proof of change. The system investigates possible cloud/haze, illumination or radiometric problems, registration failure and processing inconsistency. This prevents a systematic artifact from becoming a high-confidence "change event."

### 6.4 Seasonal false alarms in two-date mode

Seasonal rejection normally relies on history, which a single before/after pair does not have. Since seasonal change is likely the largest false-alarm source in Indian AOIs, the MVP handles it explicitly:

1. **Same-season pair selection.** Where the archive allows, pairs are chosen from the same time of year (for example March vs March), so phenology is not the main difference.
2. **Seasonal pairs as negatives.** Seasonal pairs are included as **no-change** examples when training and validating the evidence model and thresholds.
3. **Soft cropland handling.** Vegetation-only evidence over agricultural land receives reduced evidence strength but is never automatically rejected (section 6.7).
4. **Separate reporting.** The false-positive rate on seasonal pairs is reported as its own number in the ablation.
5. **Temporal narrowing is a separate, gated question.** Intermediate scenes used to narrow the date interval come from other seasons and are exposed to the same phenology effect. Section 7 compares each with both endpoints as a proposed safeguard, and section 16.5 tests whether it works.

**Hypothesis status.** The two-endpoint comparison used for temporal narrowing is a **proposed safeguard against seasonal false narrowing**, not a guaranteed one. It is validated empirically before being relied upon (section 16.5), and until it passes narrowing stays disabled.

The final report states which of these were actually used.

### 6.5 Snow, shadow, illumination and view geometry

The brief names snow, shadows, illumination and view-angle differences, and sensor differences, as confounders. The MVP handles them as follows:

- **Snow and ice.** Snow/ice pixels (SCL snow class, checked with a snow index) are treated as invalid, not as change. Pairs with heavy snow cover are flagged. Snow-on to snow-on or snow-off to snow-off pairs are preferred. This matters most for high-altitude or mountainous AOIs.
- **Shadows.** Cloud shadow comes from the SCL/geometric mask. Terrain shadow shifts with sun angle, so same-season pairing is the first defence. An optional DEM-based shadow/illumination mask is a stretch item, and any DEM must be pre-staged with its licence declared.
- **Illumination.** Surface reflectance plus per-pair normalisation, with the sun-elevation difference stored as a quality feature.
- **View geometry.** Pair scenes from the **same relative orbit** where the archive allows, so viewing geometry is comparable. Relative orbit and view angles are stored as quality features. This shrinks the choice of pairs, so the spike measures how many usable pairs remain.
- **Sensor differences.** The MVP is single-sensor (Sentinel-2), with satellite-unit and processing-baseline differences handled at ingest (section 5.1). Cross-sensor comparison needs a documented band adjustment and is a stretch item.
- **Evaluation.** If the AOI contains snow or high relief, include such pairs in the false-alarm set and report them separately.

### 6.6 Candidate generation and candidate-rate measurement

Candidates come from a union of embedding distance and ΔNDVI / ΔMNDWI / ΔNDBI signals. The union is kept only if measurement shows it helps. For every experiment record total pixels/patches, candidates generated, candidate percentage, candidates per km², downstream event count and processing time, and compare **embedding only**, **spectral only** and **union** (section 16.12). If the union produces too many candidates for the compute budget, tighten thresholds or drop a signal.

### 6.7 Cropland handling

Vegetation-only evidence over agricultural land receives **reduced evidence strength**, but is **never automatically rejected**. Structural, spectral, spatial or embedding evidence can restore confidence. This prevents genuine construction or clearance on farmland from being suppressed. The effect is measured on agricultural negatives (F8) and on positives that lie over cropland, so that missed real changes are visible in the evaluation, not only avoided false alarms.

---

## 7. Change Pipeline

### Stage A: Candidate detection (loose and cheap; Tier 0)

- Embedding distance between observations, converted to a robust z-score or percentile against the location's own history where sufficient history exists, or against scene-level statistics otherwise.
- ΔNDVI, ΔMNDWI and ΔNDBI using robust/adaptive thresholds.
- Signals computed on valid pixels only.
- A **union of signals** selects candidates. Signal strengths and the number of agreeing signals are retained as evidence features.
- Tiles failing quality gates are skipped or down-weighted.
- Per-tile thresholds use robust statistics with variance floors, to avoid unstable thresholds when variation is very small.
- **The union is measured, not assumed to help.** Candidate rate and single-signal ablations are required (section 6.6 and 16.12).

### Stage B: Evidence model and typing (Tier 0)

See section 8.

### Stage B1: Bracketing interval (Tier 0)

Every change event gets an interval from the pair itself:

- `T_last_unchanged`: the before observation.
- `T_first_changed`: the after observation.
- `interval_days`: the gap between them, the widest and most honest answer.
- `search_mode = pair_only`, no persistence claim.

This is the required baseline for the brief's "earliest available observation at which the change is supported." It works with any before/after pair and needs no extra scenes.

### Stage B2: Controlled temporal boundary search (Tier 2, gated)

If clean intermediate scenes exist between the two dates, they may tighten the interval. Three difficulties are handled explicitly: (a) intermediate scenes are from other seasons, so a plain "changed vs the before scene" test would fire on phenology; (b) bisection implicitly assumes a roughly monotone sequence and can miss a change-then-revert pattern; (c) a search that tests only some scenes cannot claim to have found the earliest one.

> **The narrowing is disabled by default.** It is enabled only after the false-narrowing experiment (section 16.5) meets criteria declared beforehand. Until then, every event keeps the bracketing interval from Stage B1.

**Per-scene classification (the nearer-state test).** For an eligible intermediate scene S, compute the evidence features of S against the before scene B and of S against the after scene A. S is **before-like** if clearly closer to B, **after-like** if clearly closer to A, and **ambiguous** otherwise. Comparing with both endpoints is a **proposed safeguard against seasonal false narrowing**: a season effect that moves S away from B should also move it away from A, and tends to give an ambiguous label instead of a false "changed" label. This is a hypothesis. It is validated empirically before being relied on (section 16.5), and it is not described as a guarantee anywhere in the deliverables.

**Procedure**

```text
Eligible clean observations at the location
              ↓
Coarse temporal scan  (evenly spaced subset, about k_coarse scenes)
              ↓
Candidate transition window  (between last before-like and first
                              after-like coarse scene, padded one step each side)
              ↓
Dense boundary refinement  (test every eligible scene in the window)
              ↓
Consistency check over all tested scenes
              ↓
      consistent          inconsistent
          ↓                    ↓
   narrow interval     flag ambiguous, keep bracket
```

1. **Eligibility.** Only scenes that are clean _at the candidate location_ are used (not just clean across the tile). Each goes through the same quality, snow/shadow and registration checks as the main pair.
2. **Coarse scan.** Classify an evenly spaced subset of eligible scenes (proposed default: up to 8).
3. **Transition window.** Take the span between the last coarse before-like scene and the first coarse after-like scene, and widen it by one coarse step on each side.
4. **Dense refinement.** Classify every eligible scene inside the window.
5. **Consistency check.** Ignoring ambiguous scenes, the tested sequence must be before-like scenes followed by after-like scenes, with no flip back. In addition, the fraction of ambiguous scenes must not exceed a declared limit (proposed default: 25%), and coarse scenes outside the window must agree with the nearer endpoint.
6. **Result** is one of three modes.

**Search modes**

| Mode                             | When                                                                                  | What is reported                                                                                                                                                                            |
| -------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **A: Exhaustive**                | Number of eligible scenes is small (proposed default: 10 or fewer), so all are tested | "Earliest supported observation **within the tested archive**"                                                                                                                              |
| **B: Refined search**            | Larger archives: coarse scan plus local refinement                                    | "Earliest supported observation **found by the configured search procedure**"                                                                                                               |
| **C: Non-monotonic / ambiguous** | Evidence flips (for example `B B A B A A`) or too many scenes are ambiguous           | **No precise earliest change is claimed.** Status `ambiguous temporal sequence`, the tested dates and labels are shown, the bracketing interval is kept, and the event goes to Needs review |

Mode B never claims to have found _the_ earliest observation, only what the configured procedure found, because scenes between coarse samples outside the refinement window were not tested.

**Interval rule (Modes A and B).**

- `T_last_unchanged` = latest before-like scene such that no earlier tested scene is after-like.
- `T_first_changed` = earliest after-like scene such that no later tested scene is before-like.

### Date-pinning quality (categorical, not a probability)

`date_pinning_quality` takes one of four values:

| Value       | Typical conditions (thresholds declared before evaluation)                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------ |
| `high`      | Mode A, or Mode B with full refinement coverage; consistent sequence; at most one ambiguous scene; narrow interval |
| `medium`    | Consistent sequence but partial coverage, a few ambiguous scenes, or a moderately wide interval                    |
| `low`       | Bracketing interval only, or few intermediate scenes, or many ambiguous scenes                                     |
| `ambiguous` | Mode C: inconsistent sequence                                                                                      |

It is based on the number of clean intermediate scenes, search coverage, consistency, interval width, quality and persistence evidence where available. It is not the probability that a calendar day is correct.

### Stage C: Temporal validation, persistence and seasonal baseline (Tier 2)

Applies only where a clean observation series exists for a candidate.

**Seasonal baseline.** Use historical clean observations grouped by an appropriate seasonal window instead of assuming the same calendar month always represents the same state. For Indian AOIs, account for agricultural cycles, monsoon-driven vegetation changes, and seasonal water expansion and contraction. Where history is insufficient, report `insufficient_data` instead of pretending a reliable baseline exists.

**Break test.** Compare pre- and post-change windows against the seasonal baseline and measure break magnitude, pre-break variability, post-break stability, and agreement with spectral/embedding evidence.

**Persistence.** Defined using **clean confirming observations**, not nominal satellite revisit dates. States:

- `persistent`
- `pending_confirmation`
- `seasonal_rejected`
- `transient`
- `insufficient_data`

### Dating output fields (used in both modes)

| Field                                                               | Meaning                                                 |
| ------------------------------------------------------------------- | ------------------------------------------------------- |
| `T_last_unchanged`                                                  | Last clean observation consistent with the before state |
| `T_first_changed`                                                   | First clean observation supporting the changed state    |
| `interval_days`                                                     | Gap between these observations                          |
| `date_source`                                                       | Sensor and scene ID supporting each date                |
| `search_mode`                                                       | `pair_only`, `exhaustive`, `refined` or `ambiguous`     |
| `n_intermediate_eligible` / `n_intermediate_tested` / `n_ambiguous` | How much intermediate evidence was available and used   |
| `search_coverage`                                                   | Share of the eligible scenes actually tested            |
| `temporal_consistency`                                              | `consistent`, `inconsistent` or `not_applicable`        |
| `date_pinning_quality`                                              | `high`, `medium`, `low` or `ambiguous`                  |

User-facing language is **"earliest supported observation found under the configured search procedure"** (or "within the tested archive" for Mode A), never "exact event date."

### Two-date behaviour

If only two usable observations exist:

- run the quality and registration checks
- use spectral and embedding evidence
- use the evidence model
- apply the same-season handling in section 6.4
- record the before and after dates as `T_last_unchanged` and `T_first_changed` (`search_mode = pair_only`, `date_pinning_quality = low`) and mark persistence as **unavailable** rather than assuming it

---

## 8. Evidence Model and Typing

### Evidence model (Tier 0)

The MVP learned component is a **small, calibrated model on evidence features**, not a deep network.

- **Model:** logistic regression by default, or a shallow gradient-boosted model with monotonic constraints, chosen by grouped cross-validation.
- **Features (per candidate patch):** embedding distance (robust z-score); ΔNDVI / ΔMNDWI / ΔNDBI summaries (median, and fraction of valid pixels beyond adaptive thresholds); number of agreeing signals; valid-pixel fraction and cloud/snow/shadow fractions; registration residual; sun-elevation, relative-orbit and processing-baseline differences; spatial-coherence features; scene-level candidate rate; **local historical variability** (local pre-change variance, historical index variability, normalised change magnitude, local temporal variability).
- **Local historical variability** distinguishes a large change in an inherently unstable area from the same change in a historically stable one. It is computed from other clean archive scenes at the location when enough exist (proposed: at least 6). Otherwise it is marked missing with an explicit indicator, not imputed as "stable."
- **Target:** binary, real change vs no change (section 15).
- **Why not a deep network first.** With a few hundred to a couple of thousand labelled patches, a deep network has more capacity than the data supports, is harder to calibrate and hides why it fired. A small model calibrates more reliably, exposes feature contributions for the evidence packet, and can be validated with grouped cross-validation.
- **Training data:** own-AOI patches (section 15), plus public patches (for example derived from OSCD) if licence and fit allow. Results on own-AOI held-out data are reported separately.

### Change network (Tier 2)

A lightweight Siamese network (FC-Siam / SNUNet-class) or a frozen remote-sensing encoder with a small change head can be trained on public Sentinel-2 change data (OSCD if its terms allow). Its output is added as one extra feature to the evidence model and kept only if it improves grouped cross-validation results. Architecture choice must be justified by measured results and compute cost. Public datasets do not cover every change category, and OSCD is small (about two dozen image pairs, urban-focused).

### Geographic generalisation

Splits are grouped by scene or AOI, never by tile. Patches from the same scene are correlated, so a random patch split leaks information and inflates results.

The final report states:

- the geographic split method
- scenes/AOIs in each split
- whether temporal neighbours were separated
- number of samples in each split

### Change typing

Change type is derived from index signatures, transitions and spatial patterns, with an explicit `other/unknown` class.

The MVP typing is primarily rule-based, so the system reports a **type evidence score / rule agreement**, not a calibrated probability. A calibrated per-type probability is future work unless enough labelled examples are collected.

| Type             | Typical evidence                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| Clearance        | NDVI down, bare-soil signal up, built-up signal not strongly up                                |
| Construction     | NDVI down, built-up signal up; persistent with limited recovery _(when multi-date data exist)_ |
| Water gain/loss  | MNDWI change consistent with historical water behaviour _(where history exists)_               |
| Road development | Thin, linear, connected pattern; lower reliability at 10 m                                     |
| Other/unknown    | Evidence indicates change but does not support a reliable known type                           |

Index rules are **features and soft consistency checks**, not unconditional vetoes. Conflicting evidence lowers confidence and is shown to the analyst. Typing is evaluated as described in section 16.13 and is not a primary success criterion when labels are weak.

### 8.1 Baseline and ablation ladder

One ladder is used everywhere (sections 16.3 and 16.8). Each rung adds one component to the previous one, so the effect of each component can be attributed.

| ID  | Configuration                                                                                             |
| --- | --------------------------------------------------------------------------------------------------------- |
| A0  | Raw / spectral difference baseline (simple index differencing on valid pixels)                            |
| A1  | + quality masking (cloud, shadow, snow)                                                                   |
| A2  | + registration handling (measure-first conditional correction)                                            |
| A3  | + seasonal handling (same-season and same-orbit pairing, seasonal-pair negatives, soft cropland handling) |
| A4  | + embedding evidence                                                                                      |
| A5  | + evidence model                                                                                          |
| A6  | + spatial context                                                                                         |
| A7  | + calibrated confidence and frozen operating point (full system)                                          |

The question the ladder must answer: **which component actually produces the improvement?** Rungs A1–A4 use simple rules with thresholds set on development data. Every rung's configuration is frozen on development data before any held-out scoring.

---

## 9. Evidence Fusion, Confidence and Thresholding

A single logistic regression is not presented as producing three unrelated probabilities. The system separates:

### A. Probability of real change

Produced by the evidence model (section 8). Inputs may include:

- change-network probability _(Tier 2, if built)_
- embedding distance
- spectral change magnitude
- local historical variability
- quality
- number of agreeing signals
- spatial coherence
- registration quality
- persistence _(multi-date mode only; absent inputs are handled explicitly, not imputed as "good")_

A separate two-date model or explicit handling of missing factors keeps the score valid in both modes.

### B. Type evidence score

A rule-agreement score summarises how strongly the observed evidence supports the assigned type. It is not presented as a calibrated probability unless a sufficiently labelled validation set exists.

### C. Date-pinning quality

The categorical `date_pinning_quality` from section 7 (`high`, `medium`, `low`, `ambiguous`). It is not a probability.

### Pipeline: fitting, calibration, threshold, freeze, one evaluation

Fitting, calibration, threshold selection and final evaluation are separate stages. Nothing learned or chosen at one stage may look at data from a later stage.

```text
DEVELOPMENT DATA (grouped by scene / AOI)
        ↓
Grouped cross-validation
        ↓
Evidence-model fitting  →  out-of-fold predictions
        ↓
Calibration  (fitted on out-of-fold predictions, itself grouped)
        ↓
Threshold selection  (rules fixed in advance, section below)
        ↓
FREEZE CONFIGURATION  (evaluation_config.yaml)
        ↓
FINAL HELD-OUT DATA  (untouched until now)
        ↓
ONE EVALUATION
```

- Folds are grouped by scene or AOI, never by tile.
- The held-out data are used to evaluate the frozen system once. They are **not** used to tune, select or re-select anything: not the model, calibrator, threshold, MMU, τ, search parameters or target.
- If a bug is found after the held-out evaluation, the fix is recorded as a new version, the affected held-out results are reported as such, and fresh held-out scenes are used if available.

### Calibration: compare, do not assume

Platt scaling is not assumed to be necessary. Compare, on grouped validation:

| ID  | Method                                  |
| --- | --------------------------------------- |
| C0  | Raw logistic probability                |
| C1  | Logistic + Platt calibration            |
| C2  | Shallow gradient boosting + calibration |

Isotonic regression is avoided at this data size because it overfits. Choose by grouped validation using the Brier score, reliability diagram and calibration error (with few bins and a clustered bootstrap interval where appropriate).

### Precision target and threshold: a protocol fixed in advance

The target is not chosen after looking at the final test. Before the held-out evaluation, the following are defined and frozen:

1. **Target ladder:** 95, 90, 85, 80 (percent).
2. **Minimum evidence requirement:** a minimum number of reported events and a minimum number of independent scenes/AOIs behind the threshold decision (proposed defaults: the event count from the table below, and at least 10 independent scenes/AOIs).
3. **Threshold-selection rule:** on the development out-of-fold predictions, the threshold is the lowest confidence score at which the **scene/AOI-clustered bootstrap lower bound** of precision meets the target. The target is the highest ladder rung for which the minimum-evidence requirement is also met.
4. **Uncertainty method:** scene/AOI-clustered bootstrap (one-sided lower bound, 95%). See section 16.10.
5. **Tie-breaking and fallback:** the rule for ties, and the status assigned when no rung qualifies.

**Minimum events: a necessary condition only.** If every reported event is correct, the 95% lower bound on precision by the Wilson formula is n / (n + 3.84), where n is the number of reported events. This shows the smallest number of reported events that a target could ever certify, even with zero errors:

| Target precision | Minimum reported events, all correct |
| ---------------- | ------------------------------------ |
| 80%              | 16                                   |
| 85%              | 22                                   |
| 90%              | 35                                   |
| 95%              | 73                                   |

Errors raise these numbers, and events from one scene are correlated, so the real requirement is higher. The table is a **sanity check that catches an impossible target early**. Wilson is **descriptive only and is never used to certify a result or select a threshold.**

**Status reported for the frozen system on the held-out data**

| Status                  | Meaning                                                                                                                        |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `certified`             | Clustered-bootstrap lower bound on held-out precision meets the target, with the minimum events and independent scenes present |
| `provisional`           | Observed precision meets the target, but the interval or the number of independent scenes is not sufficient for a strong claim |
| `insufficient_evidence` | Fewer events or independent scenes than the declared minimum, so no claim is made                                              |
| `not_met`               | Observed held-out precision is below the target                                                                                |

If there are too few independent scenes for stable inference, results are reported as **indicative** with per-AOI numbers. The system never falls back to Wilson to make a claim.

The system uses three bands (**Report**, **Needs review**, **Reject**), and the final report states the measured precision/recall trade-off instead of claiming a universal threshold.

---

## 10. Spatial Context and Event Formation

For each candidate/event calculate:

- `num_neighbor_candidates`
- `mean_neighbor_score`
- connected components
- minimum mapping unit
- event area
- spatial compactness / coherence where useful

Neighbour evidence is a **mild adjustment**, not a hard gate, so isolated but important changes are not automatically discarded.

### Event merging

Adjacent candidate tiles are merged into one event before ranking, so the analyst queue is not filled with dozens of tiles for the same physical location. Each event retains links to constituent tiles, source observations, candidate scores, evidence features and processing versions, and a **footprint geometry and change mask** used by the aggregation layer (section 16.11).

---

## 11. Discovery and Clustering

### Find similar sites

An analyst selects a tile or event and requests nearest neighbours in embedding space within a chosen AOI, a date range and optional quality constraints. This turns retrieval into a discovery tool rather than only a benchmark feature.

### Clustering (Tier 1: simple pass first)

Run k-means or HDBSCAN once per AOI over tile (and event) embeddings. Show clusters on the map with representative chips, and let the analyst jump from an event to its cluster. Clusters are **discovery aids**, not ground-truth categories, and are checked with a simple internal metric (for example silhouette) and visual inspection.

Clustering is **secondary to retrieval** and is evaluated after it. Without ground-truth cluster labels, clusters are described qualitatively or with unsupervised metrics, and no clustering accuracy is claimed (section 16.13).

---

## 12. Analyst Workflow and Provenance

### Review queue

Each event displays:

- before/after imagery
- change-mask overlay
- change type
- P(real change)
- type evidence score, where available
- factor/evidence breakdown
- source scenes, acquisition dates, location
- processing/model versions
- quality warnings

Every event also shows the earliest-supported-observation interval with search mode, search coverage and temporal consistency, the date-pinning quality, and the tested before-like / after-like / ambiguous sequence. **When multi-date data exist**, it shows the time-series plot with baseline and break.

### Analyst actions

Confirm, reject, correct type, add notes, add tags, set priority.

### Audit trail

Stored per decision:

- analyst decision, timestamp, user ID, event ID
- model versions and preprocessing versions
- threshold/configuration version
- source scene IDs

### Feedback policy

Analyst feedback is stored as labelled data and used in two ways.

**Session-level reranking (Tier 1; the first item cut under schedule pressure, and it must not delay detection or evaluation).** After a confirm or reject, the queue reranks. Candidates similar to confirmed events (by embedding and evidence features) move up, and those similar to rejected events move down. The adjustment is bounded, logged in the audit trail, reversible, and labelled in the UI (for example "boosted: similar to confirmed event #12"). The calibrated confidence, thresholds and production model are **not** changed, and rank score and confidence are shown separately.

**Offline refinement.** The production model and thresholds are never modified live. Instead:

> feedback → versioned dataset → offline retraining/calibration → held-out evaluation → deployment

### Provenance export

GeoJSON, CSV and a PDF summary. Every exported event retains source-scene IDs, processing steps, model versions and evidence.

### Structured summaries

Templates based on stored fields, for example:

> "Construction-type change, approximately X ha, earliest supported observation D2, last unchanged D1, change confidence C, source scenes S1/S2."

Every statement is traceable to a stored value.

---

## 13. Incremental Indexing and Scale

Incremental ingestion is an explicit capability.

```text
NEW SCENE
   ↓
Validate metadata
   ↓
Quality mask / preprocess
   ↓
Tile
   ↓
Generate embeddings
   ↓
Append vector IDs
   ↓
Write/update metadata
   ↓
Available for search
```

The implementation maintains stable tile IDs, a vector-to-tile ID mapping, metadata records, scene version information and a replacement/deletion policy. Index checkpointing and a recovery procedure are **optional engineering hardening**.

The exact FAISS index type (HNSW/IVF/etc.) is selected after benchmarking insertion, search latency, memory and rebuild behaviour. Some index types do not support deleting vectors and some need training on representative data, so the replacement/deletion policy is tested against the chosen type (for example by marking vectors as inactive in the metadata database and filtering them, with periodic compaction).

**Demonstration requirement:** existing archive → ingest one new scene → process → append → query returns the new scene without a full archive rebuild.

---

## 14. Infrastructure and Offline Operation

### Stack

Python, PyTorch, rasterio/GDAL, xarray or Zarr where useful, FAISS, PostgreSQL + PostGIS or SQLite + SpatiaLite, FastAPI, Leaflet or an equivalent web-map front end.

### Offline by design

All of the following are staged locally: imagery, model weights, tokenizers/preprocessors, land-cover layers, vector indexes, metadata database and application dependencies.

The system is tested with **network access disabled well before the final demo**, not only immediately before submission.

### Sovereign / on-premises design

- data need not leave the deployment environment
- model inference runs locally
- indexes stay local
- analyst actions and provenance stay local
- the solution operates without external APIs

The submission says **"designed for on-premises/offline operation"** unless actual deployment with the organiser has occurred.

### Hardware

The MVP is benchmarked on the actual demo hardware. A CPU-only fallback for core functions is optional and reported only if measured.

### Offline restart test

Before the final demo, run this sequence and record the result:

```text
network enabled
      ↓
warm up everything (load models, build caches, open the index)
      ↓
disable network
      ↓
restart the application (full process restart)
      ↓
execute the complete workflow
```

Verify that retrieval works, change detection works, model loading works, incremental indexing works, and that **no hidden network calls occur** (check with a network monitor or firewall log, not only by observing that nothing fails). Repeat well before the final demo, not only immediately before submission.

---

## 15. Data and Training Plan

| Need                            | Source / approach                                                                                                                      |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Archive for demo                | Sentinel-2 L2A over an Indian AOI, pre-downloaded                                                                                      |
| Change-model training           | OSCD as a candidate initial Sentinel-2 benchmark, used only after confirming licence/access terms, task fit and geographic limitations |
| Land cover                      | ESA WorldCover or similar, pre-staged offline (stretch)                                                                                |
| Confidence/threshold validation | Hand-labelled change/no-change set from the team's own AOIs                                                                            |
| Retrieval evaluation            | Team-created text and image queries with explicit relevance judgements, including differently phrased queries                          |
| False-alarm evaluation          | Curated examples of seasonal, cloud/haze, snow, shadow and registration effects                                                        |

**OSCD dependency gate.** OSCD is reported to be a small Sentinel-2 change-detection dataset (24 image pairs, 2015–2018) with binary change labels, oriented mainly toward urban change. Sources differ on whether labels for all pairs are public. Confirm its licence and access terms yourself before packaging or redistributing it, including the terms for the change labels. If the terms or task fit are unsuitable, use another appropriately licensed public dataset, or use OSCD only for benchmarking where permitted.

### Own-AOI labelled set

This is a major project cost, so it starts small.

**Unit of labelling: patches within pairs, not whole pairs.** Whole-pair labels give too few samples. One hundred to two hundred pairs yield only a few dozen positives, and a precision estimate from that is very noisy. Label candidate patches inside each pair instead, aiming for several hundred to a couple of thousand patches and, if possible, 100 to 150 or more positives.

**Independent scenes/AOIs are the primary unit of generalisation and of uncertainty.** A large patch count must never hide a small scene count. Patches from one scene are correlated, so the real sample size for uncertainty is the number of independent scenes and AOIs. Where labelling effort is limited, prefer **more distinct scenes with fewer patches each** over many patches from few scenes.

**Binary target**

1. real change
2. no change

**Artifact tags** (multi-select, mainly on negatives and on false candidates), using the taxonomy in section 6.2: F1 seasonal vegetation, F2 cloud/haze, F3 snow, F4 terrain/shadow/illumination/view-angle, F5 registration, F6 processing baseline, F7 water fluctuation, F8 agricultural cycle, F9 mixed/unknown. A seasonal pair or an artifact-caused difference is a _no change_ patch with the matching tag.

**Optional type tag** on positives, only if time allows: construction, clearance, water, road/linear, other/unknown.

**Sampling strata.** Label patches from each of these, and record the stratum of every patch:

```text
positive candidates
+ random negatives (non-candidate patches, so recall is not overestimated)
+ hard negatives (pipeline-flagged patches that are not real change)
+ seasonal negatives
+ atmospheric negatives (cloud, haze, snow)
+ registration negatives
+ agricultural negatives
```

**Dataset report.** Every dataset report states: total patches, positive patches, negative patches, **independent scenes**, **independent AOIs**, geographic regions, and temporal span, per split.

**Splitting.** All patches from a scene stay in the same fold. Keep one AOI or a set of scenes untouched as the final test. Record the number of independent scenes and AOIs in every fold.

### Portability to an organiser-defined AOI

The brief says the demonstration runs over an organiser-defined area and time span, and change analysis is scored on held-out cases the team has not seen. Own-AOI calibration may not transfer, so:

- **AOI-agnostic pipeline.** No AOI-specific constants. Region rules live in configuration files. A single config-driven command ingests a folder of GeoTIFF/COG scenes into tiles, embeddings and index, and the ingest time for a new AOI is measured and reported.
- **Diverse own labels.** Label pairs from at least two or three different AOIs (for example an agricultural/monsoon area, an urban-growth area, and, if possible, a high-relief or snow-affected area). Validate leave-one-AOI-out.
- **Scene-adaptive thresholds.** Thresholds are expressed relative to robust scene-level statistics instead of absolute values, so they adapt to a new scene.
- **Recalibration only with labels.** If the organisers provide a small labelled development set, recompute the operating point on it using the procedure in section 9. If not, use the conservative default operating point from leave-one-AOI-out validation and state that it was not recalibrated. The system does not claim recalibration without labels.
- **Safety net.** The scene-wide artifact check (section 6.3) runs on every new scene.

---

## 16. Evaluation Plan

Evaluation is the center of the project. The core evaluation must answer:

1. Can the system find relevant imagery, and how much of that is due to the chosen model rather than a simple baseline?
2. Can the **two-date pipeline** distinguish real change from false alarms, and which component produces the improvement?
3. Does the system keep an honest operating point, with uncertainty measured over independent scenes?
4. Does temporal narrowing avoid false narrowing, and only then does it add value?

All rungs of every comparison are **frozen on development data and then evaluated once** on the held-out data, so comparisons are fair and the held-out set is never used for tuning (section 9).

### 16.1 Retrieval (Tier 0)

- Compare R0, R1, R2 (section 5.2).
- Metrics: Recall@K, Precision@K, mAP where the query set is large enough, query latency and p95 latency, storage/index size.
- Report **seen** concepts and phrasing separately from **held-out** concepts and phrasing.
- **Robustness test:** exact query, paraphrase, synonym, unseen wording (section 5.2). Report the gap between seen and unseen groups.
- Text and image queries are evaluated separately.

### 16.2 Two-date change detection (Tier 0; primary change benchmark)

For held-out labelled patches (held out by scene/AOI), scored through the aggregation layer (section 16.11) at **pixel, tile and pair/event** level:

- precision, recall, F1, PR-AUC where appropriate
- confusion matrix and prevalence (share of positives)
- scene/AOI-clustered intervals (section 16.10) and per-AOI results
- false-positive rate / false positives per area where the data support it
- candidate/event rate at the selected operating point
- **strict** (Report only) and **lenient** (Report + Needs review) variants

MMU and τ are chosen on development data and **frozen before** held-out scoring (section 16.11). This benchmark must not depend on persistence, seasonal history, exact event dates or multi-date break detection.

### 16.3 False-alarm evaluation (Tier 0)

Every false-alarm experiment reports performance **by category** (taxonomy F1–F9, section 6.2), not as one overall false-positive number. The mandatory table (false-positive rate on no-change patches of each category; ladder steps from section 8.1):

| False alarm            | A0 baseline | A1 + quality | A2 + registration | A3 + seasonal | A7 full |
| ---------------------- | ----------: | -----------: | ----------------: | ------------: | ------: |
| F1 Seasonal vegetation |             |              |                   |               |         |
| F2 Cloud / haze        |             |              |                   |               |         |
| F3 Snow                |             |              |                   |               |         |
| F4 Terrain / shadow    |             |              |                   |               |         |
| F5 Registration        |             |              |                   |               |         |
| F6 Processing baseline |             |              |                   |               |         |
| F7 Water fluctuation   |             |              |                   |               |         |
| F8 Agricultural cycle  |             |              |                   |               |         |
| F9 Mixed / unknown     |             |              |                   |               |         |

Include the number of patches and independent scenes behind each cell, and leave a cell empty (not zero) where a category has too few samples. Also report the number and share of candidates removed by each quality/evidence stage.

### 16.4 Calibration and precision target (Tier 0)

Report for the frozen system:

- C0 / C1 / C2 comparison on grouped validation: Brier score, reliability diagram, calibration error
- the frozen protocol (target ladder, minimum evidence, selection rule, uncertainty method) and its file version
- for the selected operating point:

| Field                     |                                                                   |
| ------------------------- | ----------------------------------------------------------------- |
| target                    | chosen ladder rung                                                |
| threshold                 | frozen value                                                      |
| predicted positives       | count                                                             |
| observed precision        | held-out point estimate                                           |
| recall                    | held-out point estimate                                           |
| interval                  | scene/AOI-clustered bootstrap                                     |
| independent scenes / AOIs | counts                                                            |
| status                    | `certified` / `provisional` / `insufficient_evidence` / `not_met` |

- size of the `needs_review` band

Type evidence is not reported as a calibrated probability unless enough labels exist to justify it.

### 16.5 False-narrowing experiment (mandatory) and temporal interval evaluation

**The false-narrowing experiment is mandatory and gates Stage B2.** Temporal narrowing (section 7) stays disabled unless it passes.

**Scenarios**

```text
1. No real change      stable location, several years of clean scenes,
                       including full seasonal cycles
2. Seasonal only       vegetated / agricultural / water-varying locations
                       with no structural change
3. Change → revert     temporary disturbance or synthetic before-after-before
                       sequences, to attack the monotonicity assumption
4. Single real change  (where a real event with known timing exists, or a
                       constructed sequence with a known step)
```

For scenarios 1 and 2, construct before/after pairs from the series (same-season and cross-season) and run the full temporal search. Scenario 3 tests specifically whether the search misses an earlier or transient change.

**Required outputs**

- **false-narrowing rate:** how often the search declares a narrowed transition where there is no real change
- **ambiguous-sequence rate**
- **interval width** distribution (days)
- **missed-transition rate:** how often a real transition is missed or the interval excludes it (scenarios 3 and 4)
- number of locations, series and scenes behind each figure

**Gate criteria** are declared before the experiment is run (proposed defaults: false-narrowing rate at or below 5% on scenarios 1 and 2, and at least 80% of scenario 3 sequences flagged ambiguous). If the criteria are not met, narrowing stays disabled, every event keeps the bracketing interval, and the report says so. With few series, results are described as indicative.

**Temporal interval evaluation**

- If real change dates are available: interval containment, interval width, earliest-supported-date error, ambiguous rate.
- If real dates are not available: **do not invent a temporal accuracy metric.** Report interval consistency and false narrowing instead.
- Manually established temporal cases are used only if the team builds them, with a description of how the dates were established.

### 16.6 Analyst workload (optional)

If conducted: events reviewed, confirmed events found, time per confirmed event, false alarms reviewed, and queue size at the selected operating point.

### 16.7 System metrics

**Tier 0 / required by the brief:** indexed area, number of scenes and tiles, **index build time**, storage footprint, average and p95 query latency, hardware used, peak memory estimate.

**Optional:** incremental-ingestion time, change-analysis latency, CPU-only fallback performance, checkpoint/recovery behaviour.

Optional metrics are not promised in the selection deck unless actually measured.

### 16.8 Ablation study

The component ladder is defined in section 8.1 (A0–A7). It is a formal experiment, reported on development out-of-fold results, with every rung also scored once on the held-out data using configurations frozen on development data.

Additional required comparisons:

| Comparison                                                 | Purpose                                                           |
| ---------------------------------------------------------- | ----------------------------------------------------------------- |
| Registration: none vs measure-first conditional correction | Effect of alignment handling, especially on F5                    |
| Candidates: embedding only vs spectral only vs union       | Whether the union is useful (section 16.12)                       |
| Aggregation: pixel only vs tile vs event/pair              | Effect of aggregation (section 16.11)                             |
| Baseline correction: raw vs corrected                      | Effect of processing-baseline handling (F6)                       |
| Session-level reranking on/off                             | precision@k of the review queue after simulated feedback (Tier 1) |

Optional rows, if built: change network as an extra feature, seasonal baseline, persistence, SAR agreement. Only rows that were actually run are reported. Ablation differences are described as indicative when they are smaller than the paired scene-level interval.

### 16.9 Geographic generalisation and leakage control

Splits are grouped by scene/AOI. Where feasible:

```text
Region A → training / fitting
Region B → validation / calibration
Region C → final held-out
```

At minimum, **no patch from the same scene appears in more than one split**, and temporal neighbours from the same location are kept together or separated by a documented rule. Random tile splits are not sufficient.

### 16.10 Uncertainty reporting

With a small labelled set, point estimates alone overstate what the system shows, and patches from the same scene are correlated.

- **Primary: scene/AOI-clustered bootstrap.** Resample whole scenes (or AOIs), not patches, for precision, recall, F1, PR-AUC, calibration error and ablation differences (paired).
- **Secondary: per-AOI results**, reported next to the pooled numbers, because a pooled interval can hide large differences between AOIs.
- **Descriptive only: Wilson interval.** It assumes independent samples, so it may be shown for reference but is **not used for certification, threshold selection or any claim.**
- Report the number of positives, negatives and **independent scenes/AOIs** in each split.
- If there are too few independent clusters for stable inference (proposed: fewer than about 10), make **no strong certification claim**. Report results as indicative, with per-AOI numbers.

### 16.11 Aggregation layer: pixel, tile and pair level (Tier 0)

The brief does not fix whether scoring is per pixel, per tile or per pair. A fixed, documented aggregation layer converts events into any of these, and the choice is configuration.

**Inputs.** Every event has a footprint geometry and a change mask (section 10). **Report**-band events are the system's positive predictions. **Needs review** events are counted separately: results are reported in a **strict** variant (Report only) and a **lenient** variant (Report + Needs review).

| Level        | Rule                                                                                                                                                              |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pixel        | Predicted change mask = union of change masks of predicted events, restricted to valid pixels on both dates and to events above the minimum mapping unit (MMU).   |
| Tile         | A tile is predicted positive if predicted-event area inside it is at least τ. The tile grid is the organiser's if one is given, otherwise the system's tile grid. |
| Pair / event | A before/after pair is predicted positive if it has at least one predicted event with area ≥ MMU.                                                                 |

**Parameter freeze**

```text
development set
      ↓
select MMU and τ  (with a sensitivity table)
      ↓
FREEZE
      ↓
held-out evaluation
```

The held-out data are never used to select MMU or τ. If the organisers publish an evaluation format or a development set, the layer is configured to match it. If not, all three levels are reported. A **sensitivity table** shows metrics at a few MMU and τ values, so the result does not hinge on one setting. The **aggregation ablation** reports pixel only, tile aggregation and event/pair aggregation.

Output writers: GeoJSON (event polygons), GeoTIFF change masks, and CSV per pair and per tile, all with source-scene IDs and processing versions.

### 16.12 Candidate-generation evaluation

For every experiment record:

- total pixels/patches examined
- candidates generated, candidate percentage, candidates per km²
- downstream event count
- processing time

Compare **embedding only**, **spectral only** and **embedding + spectral union**, and report whether the union improves recall or precision enough to justify its candidate volume. A candidate-explosion threshold (candidates per km² or per scene, set from the compute budget) is declared before the run. If the union exceeds it, tighten the candidate thresholds or drop a signal.

### 16.13 Typing and clustering evaluation

**Typing (where labels exist):** type accuracy, `other/unknown` rate, confusion matrix, rule-disagreement rate. Typing is not a primary success criterion when labels are weak, and the type evidence score is never presented as a calibrated probability.

**Clustering:** clustering is secondary to retrieval and is evaluated after it. Without ground-truth cluster labels, describe clusters qualitatively or with unsupervised metrics (for example silhouette). Do not claim clustering accuracy from visual impressions alone.

---

## 17. Feasibility Spike Before Full Build

The spike is the **main go/no-go gate**. Before implementing the full platform, run a small technical spike.

### Spike objective

1. Can the selected Sentinel-2 data be processed reliably for the chosen AOI?
2. How many clean observations exist per candidate location? (This decides how much of the temporal search is realistic, and whether Mode A or Mode B applies.)
3. Which embedding model works best on the actual imagery, against the simple baseline R0?
4. Does text retrieval work on the intended query vocabulary, and on paraphrased and unseen wording?
5. Does embedding distance + spectral evidence identify useful change candidates, and how many candidates does the union produce?
6. How severe are registration and seasonal false alarms, and does same-season pairing reduce them?
7. Does measure-first registration correction help, or is native alignment already acceptable?
8. What tile size gives the best quality/latency/storage trade-off?
9. Can the intended hardware handle the workload offline?
10. Does the AOI contain snow, terrain shadow or mixed relative orbits, and how many usable same-orbit, same-season pairs remain?
11. How long does it take to ingest an unseen AOI end to end?
12. How many labelled patches, positives and **independent scenes/AOIs** can realistically be produced? (This decides the precision-target ladder and the minimum-evidence requirement.)
13. Do the scenes straddle a processing-baseline change, and does the correction pass the unit test (section 5.1)?
14. Does the temporal search give sensible results on real series, and does the false-narrowing experiment pass?

### Mandatory outputs

1. retrieval benchmark (R0–R2)
2. two-date baseline result (A0)
3. false-alarm benchmark by category (F1–F9)
4. processing-baseline test result
5. registration test result
6. candidate-explosion rate
7. temporal-narrowing false-positive test
8. storage estimate
9. runtime estimate
10. memory estimate
11. labelled-scene estimate (patches, positives, independent scenes/AOIs)
12. per-Tier-0/1/2-item effort estimate (person-days and risk rating)

Also: data-quality report, recommended tile size, selected retrieval model, and a go/no-go decision for each Tier 2 and stretch component.

### Hard gate

```text
        Is Tier 0 feasible?
          /            \
        YES             NO
         ↓               ↓
   continue to       cut scope using the
   Tier 1 / Tier 2   cut order (section 3)
```

**No Tier 1 or Tier 2 implementation begins until Tier 0 feasibility is demonstrated.**

### MVP freeze gate

At the end of Phase 1, the tier assignment of every item and the cut order are written down and locked. Later changes need an explicit trade-off.

### Spike scope

Use a small Indian AOI and a limited number of scenes. Do **not** build the full UI or advanced model stack before these questions are answered.

---

## 18. Demonstration Scenario

The demo exercises multiple PS requirements through one analyst story. **It leads with the MVP path and adds multi-date validation only if the archive supports it.**

### Demo-query selection rule

Do **not** lock the demo query before the feasibility spike. The spike benchmarks a small set of realistic queries and shows which concepts retrieve reliably at the available resolution.

- **Primary query:** the strongest measured query from the spike.
- **Fallback query:** a second query less dependent on small-object semantics.
- **PS example:** "newly built structures near a river" is shown only if the benchmark shows the model and imagery support it.

### Part 1: Core MVP flow (two-date)

1. **Discover.** A text query returns ranked candidate tiles/events, with spatial and temporal filters applied.
2. **Select.** The analyst opens one candidate, and can jump to similar sites or its cluster.
3. **Verify.** The system shows before/after imagery, the change mask, the change/no-change decision, and the earliest-supported-observation interval, with search mode, search coverage and temporal consistency shown.
4. **Explain.** The evidence panel shows, where supported: built-up signal up, vegetation signal down, evidence model agrees, spatially coherent, registration acceptable, observations sufficiently clean, same-season pairing applied.
5. **Reject a false alarm.** A plausible false alarm (for example a seasonal NDVI change, a snow edge or a cloud edge) is shown being rejected or sent to "needs review", with the reason.
6. **Decide.** The analyst confirms or rejects, and the queue reranks (boosting candidates similar to confirmed events, demoting those similar to rejected ones) without changing the model.
7. **Export.** The event is exported with location, change type, confidence, evidence, source scenes, processing/model versions and the analyst decision.
8. **Update.** A new scene is ingested and becomes searchable without rebuilding the archive.

### Part 2: Temporal search and multi-date validation (only if intermediate clean observations exist and narrowing passed its gate)

Consistent sequence:

```text
Last unchanged:                 T2
First supported change:         T4
Intermediate observations tested: T3
Search:                         exhaustive (within the tested archive)
Temporal consistency:           consistent
Date-pinning quality:           high
```

Ambiguous sequence, shown as ambiguous instead of dated:

```text
T1 before-like
T2 after-like
T3 before-like
T4 after-like

Temporal status: AMBIGUOUS — non-monotonic sequence
(bracketing interval kept, event sent to Needs review)
```

Showing that the system knows when it should not overclaim is part of the demonstration. If narrowing did not pass its gate, this part shows the bracketing interval only, and says why. The time-series plot and persistence status are added when multi-date validation is enabled.

This story covers semantic retrieval, false-alarm handling, analyst workflow, provenance, offline operation and incremental indexing, plus temporal analysis where data allow.

---

## 19. Selection-Oriented Product Differentiation

The differentiator is not "we use CLIP + FAISS + NDVI." Those are implementation components. The product-level differentiation is:

### 19.1 Evidence-first change intelligence

The system connects **semantic discovery → false-alarm-controlled change verification → evidence-backed event**, with temporal verification added when the archive supports it.

The final submission presents this as a demonstrated strength only to the extent the evaluation and demo show it.

### 19.2 False-alarm controls

**Working product label:** "False-Alarm Firewall." Use it as a demonstrated differentiator only if the ablation shows the added evidence reduces false alarms. The mechanism combines, where available: quality, registration, seasonal handling, snow/shadow/view-geometry handling, spectral consistency, spatial coherence, scene-wide artifact checks, and persistence (multi-date).

If the ablation does not show a measurable reduction, call these **false-alarm controls** and do not claim a proven firewall.

### 19.3 Honest temporal intelligence

Instead of pretending to know the exact day: **last unchanged → first supported change → uncertainty interval**, reported for every event: widest for a single pair, narrower only where the controlled temporal search finds consistent evidence and its seasonal safeguard has been validated, and flagged as ambiguous where the sequence is inconsistent.

### 19.4 Analyst trust

Every result has evidence, provenance, model version, source imagery and review status.

### 19.5 Honest uncertainty

Results are reported with scene-clustered intervals, and the precision target is chosen from what the label counts can certify. The report says when the Report band is provisional. This is a differentiator only if the report actually shows it.

### 19.6 Offline sovereignty

The workflow is designed to run locally without external inference APIs.

---

## 20. Requirement-to-Evidence Mapping

Classification: **Req** = required by the brief; **T0/T1** = MVP tiers; **Exp** = experimental extension (gated); **Str** = stretch.

| Requirement                                         | Class                               | Implementation                                              | Evidence                                                                      | Failure / limitation                                                           |
| --------------------------------------------------- | ----------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Semantic retrieval                                  | Req, T0                             | Embedding search, prompt templates, R0–R2                   | Ranked imagery, R0–R2 comparison, robustness test                             | 10 m resolution limits query vocabulary; model may be RGB-only                 |
| Multimodal retrieval                                | Req, T0                             | Text-to-image + image-to-image                              | Both query modes                                                              | Image query needs comparable imagery                                           |
| Temporal change                                     | Req, T0                             | Two-date engine                                             | Held-out pair metrics at pixel/tile/pair                                      | No persistence in two-date mode                                                |
| Earliest supported change                           | Req, T0 (bracket) / Exp (narrowing) | Bracketing interval; conditional controlled temporal search | Intervals with search mode, coverage, consistency; false-narrowing experiment | Non-monotonic series flagged, not dated; narrowing off until validated         |
| False-alarm suppression                             | Req, T0                             | Quality controls + evidence model                           | False-alarm table by category F1–F9, ablation ladder                          | Depends on labelled negatives; "Firewall" label only after ablation            |
| Precision focus                                     | Req, T0                             | Frozen precision-target protocol, three bands               | Status (`certified` / `provisional` / ...) with clustered interval            | May be provisional with few independent scenes                                 |
| Similar-site discovery                              | Req, T0                             | Vector nearest neighbours                                   | Selected site → similar sites                                                 | Depends on embedding quality                                                   |
| Clustering                                          | Req, T1                             | k-means / HDBSCAN pass                                      | Cluster view, unsupervised metric, qualitative check                          | No ground-truth labels, so no accuracy claim                                   |
| Analyst workflow                                    | Req, T1                             | Review queue, confirm/reject, correct                       | Demo                                                                          | n/a                                                                            |
| Feedback use                                        | Req (where supported), T1           | Session-level reranking, model frozen                       | Queue reranks after decisions; precision@k comparison                         | First to be cut under schedule pressure; does not change calibrated confidence |
| Provenance                                          | Req, T1                             | Evidence packet, export                                     | Export with versions and source scenes                                        | n/a                                                                            |
| Incremental indexing                                | Req, T1                             | Append without rebuild                                      | New scene searchable                                                          | Deletion policy depends on index type                                          |
| Offline operation                                   | Req, T0                             | Local weights and data, restart test                        | Network-disabled restart and full workflow                                    | Hidden network calls must be checked                                           |
| Scalability and report metrics                      | Req, T0                             | Area, scenes/tiles, build time, storage, latency, hardware  | Metrics table                                                                 | Measured on demo hardware only                                                 |
| Held-out scoring                                    | Req, T0                             | Aggregation layer with frozen MMU/τ                         | Results at each level with sensitivity table                                  | Organiser granularity unknown                                                  |
| Persistence / seasonal baseline                     | Exp                                 | Multi-date validation                                       | Only if evaluated                                                             | Needs clean history                                                            |
| SAR corroboration                                   | Str                                 | Sentinel-1 agreement                                        | Optional ablation                                                             | Not required unless the organiser's data needs it                              |
| Checkpoint/recovery, CPU fallback, analyst workload | Str                                 | Optional                                                    | Only if measured                                                              | Not promised unless measured                                                   |

---

## 21. Risks and Mitigations

| Risk                                                                                   | Mitigation                                                                                                                                                                                                            |
| -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Few clean optical scenes during monsoon periods                                        | Two-date MVP does not depend on long series; `insufficient_data` and `pending_confirmation` states; SAR as stretch                                                                                                    |
| Seasonal change flagged as real in two-date mode                                       | Same-season pairing, seasonal pairs as negatives, cropland flagging, separate reporting                                                                                                                               |
| Interval narrowing lands on a seasonal date                                            | Two-endpoint nearer-state test treated as a hypothesis, same quality checks on every scene, ambiguous label, mandatory false-narrowing experiment that gates narrowing                                                |
| Temporal search misses an earlier transient change or a change-then-revert             | Exhaustive test for small n; coarse scan, dense refinement and consistency check for larger n; non-monotonic sequences flagged ambiguous and not dated; change-then-revert scenario in the false-narrowing experiment |
| Precision target cannot be certified with few events (Report band empty)               | Protocol fixed before final evaluation (ladder, minimum evidence, selection rule), statuses `certified` / `provisional` / `insufficient_evidence` / `not_met`, target never silently lowered                          |
| Interval estimates too optimistic because patches are correlated                       | Scene/AOI-clustered bootstrap as the only primary method, Wilson descriptive only and never used for claims, per-AOI results, independent-scene counts reported                                                       |
| Unknown scoring granularity (pixel / tile / pair)                                      | Fixed aggregation layer with strict/lenient variants, MMU and τ fixed before test scoring, sensitivity table                                                                                                          |
| Reflectance offset across Sentinel-2 processing baselines creates false index shifts   | Record processing baseline, convert to a common convention on ingest, check in the spike, tag as an artifact class                                                                                                    |
| Domain gap between public training data and evaluation AOI                             | Geographic holdout, own-AOI labels, multi-signal evidence                                                                                                                                                             |
| Organiser-defined AOI differs from the team's own AOI                                  | AOI-agnostic config-driven ingest, multi-AOI labels with leave-one-AOI-out validation, scene-adaptive thresholds, conservative default operating point                                                                |
| Snow, terrain shadow or view-geometry differences (especially high-relief AOIs)        | Snow/shadow masks, same-season and same-relative-orbit pairing, separate reporting of these pairs                                                                                                                     |
| Retrieval quality on 10 m imagery                                                      | Benchmark real chips early; scope queries to visible patterns                                                                                                                                                         |
| Held-out queries phrased differently from tuning queries                               | Prompt templates, offline synonym lists, evaluation on differently phrased queries                                                                                                                                    |
| Multispectral input mismatch with retrieval model                                      | Verify band support; benchmark RGB/false-colour vs multispectral-capable models                                                                                                                                       |
| Small labelled set gives wide, noisy estimates and thresholds                          | Patch-level labelling, more distinct scenes, grouped cross-validation, small calibrated evidence model, conservative operating point                                                                                  |
| Scene-wide artifacts                                                                   | Scene-wide candidate-rate QC and registration checks                                                                                                                                                                  |
| Calibration or threshold leakage into evaluation                                       | Separate stages (fit, calibrate, threshold, freeze, one evaluation); held-out data scored once after the freeze                                                                                                       |
| Geographic leakage                                                                     | Scene/AOI-level split                                                                                                                                                                                                 |
| FAISS metadata/update limitations                                                      | Separate vector index from metadata DB; define append/replacement policy and test it on the chosen index type                                                                                                         |
| Model or dataset licence / offline packaging issues                                    | Verify licence and offline loading early; keep a provenance register                                                                                                                                                  |
| Storage or runtime explosion                                                           | Benchmark tile size, storage and latency in the feasibility spike                                                                                                                                                     |
| Scope creep                                                                            | Tier 0 / Tier 1 / Tier 2 boundaries, MVP freeze gate, per-item time estimates from the spike, explicit cut order                                                                                                      |
| Automatic feedback corrupts production model                                           | Version feedback and retrain offline; session reranking is bounded, logged and does not alter confidence                                                                                                              |
| Road/vehicle claims exceed resolution                                                  | Report resolution limitations explicitly                                                                                                                                                                              |
| Candidate explosion from the union of signals                                          | Candidate-rate monitoring, single-signal ablations, declared candidate budget                                                                                                                                         |
| Too few independent scenes give false statistical confidence from thousands of patches | Independent scenes/AOIs as the primary unit, clustered inference, `insufficient_evidence` status                                                                                                                      |
| Cropland down-weighting hides real construction or clearance                           | Soft evidence reduction only, never automatic rejection, evaluation on positives over cropland                                                                                                                        |
| Unnecessary resampling creates registration artifacts                                  | Measure first, correct only when needed, measure again, with-vs-without ablation                                                                                                                                      |
| Overclaiming temporal accuracy                                                         | Bracketing interval as default, narrowing gated, "found under the configured search procedure" wording, ambiguous status                                                                                              |

---

## 22. Known Limitations

- At 10 m resolution, small objects and many roads are at or below the reliable detection scale.
- Change dates are intervals bounded by available clean observations, not exact event dates. A single pair gives the widest interval, and without persistence checks the first changed observation rests on one clean scene.
- Interval narrowing depends on having several clean intermediate scenes at the location. In refined-search mode the procedure does not claim to have found _the_ earliest observation, only the earliest one it found, because scenes outside the refinement window were not tested. Ambiguous or inconsistent sequences are left at the bracketing interval and flagged.
- The two-endpoint seasonal safeguard for narrowing is a hypothesis. It is only relied on if the false-narrowing experiment passes, and even then it is not guaranteed for every landscape.
- Snow and shadow handling depends on the SCL snow class and, if used, a DEM. High-relief or snow-covered AOIs will have more masked area and lower coverage.
- Recalibration on an unseen AOI is only possible with labels. Otherwise a conservative default operating point is used.
- With few validation events or few independent scenes, the target precision that can be certified is limited. Results may be `provisional` or `insufficient_evidence`, and the report says so. Wilson intervals are never used to close that gap.
- Scoring granularity is unknown until the organisers specify it. The aggregation layer covers pixel, tile and pair level, but results at one level do not automatically transfer to another.
- Change typing may return `other/unknown` instead of forcing a label, and type is a rule-agreement score, not a calibrated probability.
- Public change datasets may not represent all desired Indian geographic and land-use conditions.
- Land-cover-aware rules depend on the quality and suitability of the land-cover layer.
- Seasonal baselines require enough clean historical observations; without them, only same-season pairing is available.
- SAR support is a stretch goal unless the organiser's data requires it.
- Retrieval quality depends on the selected model's compatibility with the imagery and query vocabulary.
- Confidence is only as trustworthy as the calibration labels and held-out evaluation design.
- With a small labelled set, reported precision and recall carry wide confidence intervals. Results are reported with intervals, and small differences are not claimed as improvements.

These limitations are presented openly rather than hidden in the demo.

---

## 23. Build Plan

Team size and remaining time determine the calendar, so the plan is a dependency-aware sequence with gates. Tier 0 comes before everything else.

### Phase 0: Feasibility spike

Data, preprocessing, retrieval baselines, basic change, false alarms and a temporal-search trial (section 17).

**Gate:** Tier 0 is shown to be feasible, and per-item effort estimates exist.

### Phase 1: Foundation

- ingestion validation block, quality masking, tiling, metadata DB
- processing-baseline correction and its unit test
- measure-first registration
- config-driven, AOI-agnostic ingest command
- patch-level labelling workflow (binary label, F1–F9 tags, strata, independent-scene counts)
- evaluation harness with the aggregation-layer skeleton and baseline A0

**Gate (MVP freeze gate):** reproducible preprocessing and a measurable baseline scored at all three levels. Tier assignments and cut order are locked.

### Phase 2: Retrieval

Embedding model, FAISS, metadata filtering, prompt templates and synonym lists, R0–R2 benchmark, robustness test.

**Gate:** held-out retrieval evaluation works, including paraphrased and unseen wording.

### Phase 3: Two-date change

- candidate generation and candidate-rate measurement
- spectral, embedding, quality and spatial evidence
- same-season and same-orbit pairing, snow/shadow masking
- rule-based typing
- evidence model, aggregation layer, bracketing interval
- ablation ladder A0–A7 on development data

**Gate:** complete **development-set** (grouped cross-validation) evaluation of the two-date pipeline. The held-out data are not touched yet.

### Phase 4: Confidence

- C0 / C1 / C2 calibration comparison
- precision-target protocol, threshold selection, clustered uncertainty
- MMU and τ selection with sensitivity table
- **freeze** `evaluation_config.yaml`
- **one** held-out evaluation of the frozen system and all ladder rungs

**Gate:** configuration frozen and held-out results recorded with their status.

### Phase 5: Temporal extension (Tier 2, gated)

Only now:

- false-narrowing experiment (mandatory)
- if it passes: controlled temporal search, interval narrowing, persistence, seasonal baseline
- if it fails: narrowing stays disabled and the bracketing interval is reported

### Phase 6: Analyst workflow (Tier 1)

Workbench, review queue, provenance, export, clustering visualisation, session-level reranking (first to be cut).

### Phase 7: Offline and incremental hardening

- incremental ingestion demonstration
- **offline restart test** (section 14)
- hardware benchmark
- optional: checkpointing, recovery test, CPU-only benchmark

**Gate:** the demo works without network access after a restart, and a new scene can be added incrementally.

### Phase 8: Stretch

Only after Tier 0 metrics are stable: SAR corroboration, DEM-based terrain shadow handling, cluster-quality analysis, land-cover-aware thresholds, change network as an extra feature, advanced typing, feedback-driven offline retraining, additional sensors.

---

## 24. Expected Deliverables

### Required MVP deliverables

- source code
- offline installation/setup scripts
- architecture document
- ingestion and incremental-indexing procedure
- model/dataset provenance register
- database/index schema
- reproducible evaluation harness, including the aggregation layer and its configuration
- evaluation report covering retrieval, **two-date change detection at pixel/tile/pair level**, earliest-supported-observation intervals (including the false-narrowing experiment and its gate result), clustering check, false-alarm controls, confidence calibration and the chosen precision target, core system metrics (including index build time), core ablations, scene-clustered confidence intervals and the geographic split definition
- working offline demo

### Required evaluation artifacts

```text
baseline_results.csv
ablation_results.csv
false_alarm_results.csv
retrieval_results.csv
temporal_results.csv
calibration_results.csv
system_benchmark.csv
```

### Required configuration artifact

A frozen `evaluation_config.yaml` containing:

- model versions and weights identifiers
- preprocessing version (including processing-baseline handling and registration threshold)
- calibration method
- precision-target protocol and thresholds
- MMU, τ and aggregation rule
- temporal search method and whether narrowing is enabled
- dataset split definition
- random seeds

### Optional deliverables

Include only if implemented and measured:

- interval-containment evaluation against date ground truth
- analyst workload study
- incremental-ingestion timing and recovery benchmark
- peak-memory analysis
- CPU-only performance benchmark
- SAR ablation
- extended clustering evaluation
- extended change-type evaluation

### Required demo

1. semantic text query
2. image query, if implemented
3. candidate ranking
4. similar-site search and clustering view
5. two-date change result, evidence and earliest-supported-observation interval, with search mode and temporal consistency
6. false-alarm handling
7. analyst review with session-level reranking
8. provenance export
9. incremental ingestion
10. operation with network disabled

### Optional demo additions

multi-date timeline, interval-narrowing walkthrough with a flagged inconsistent sequence, SAR corroboration, analyst workload results.

---

## 25. Final Summary

The proposed system combines **semantic satellite-image discovery** with a **quality-aware change-detection pipeline** and an **analyst review workflow**, extended with multi-temporal validation where enough observations exist and the validation experiments allow it.

> **Discover → Verify → Trust**

The system distinguishes between **what it can detect, what it can verify, and what it can date**:

```text
Two-date:      detect + evidence + bracketing interval
Multi-date:    persistence + validation + conditional narrowing
Non-monotonic: flag ambiguity rather than overclaim
```

The system does not treat every image difference as a meaningful event:

> **Search finds candidates.
> Quality, seasonal and spatial checks verify them.
> Temporal evidence adds further confirmation when the archive supports it and the search has been validated.
> Evidence fusion produces a calibrated decision.
> Provenance lets an analyst understand and audit the result.**

The MVP is intentionally narrow:

> **Sentinel-2 → preprocessing → semantic retrieval → two-date change detection → false-alarm suppression → earliest-supported interval → evidence/confidence → analyst review → provenance**

Every additional component must improve a measured metric, reduce a documented failure mode, or satisfy an explicit requirement. Otherwise it stays out of the core system.

The most important proof is a working offline demonstration: the system takes a natural-language or image query, discovers relevant locations, verifies a suspected change, rejects a plausible false alarm, explains the evidence, and preserves the complete provenance of the analyst's decision. It gives an honest earliest-supported-observation interval, narrowed only where the evidence allows, shows an ambiguous sequence as ambiguous, and reports its uncertainty instead of hiding it.

---

## Appendix A: Open items and verification status

### A. Must verify externally

| Item                                                                        | Status                                                               | Action                                                                                                |
| --------------------------------------------------------------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Official SIH26227 problem statement                                         | Checked against a copy of the brief text on a third-party site       | Compare against the official portal text                                                              |
| Official deadline                                                           | A third-party tracker lists **30 September 2026**                    | **Confirm on the official SIH portal immediately**                                                    |
| Official submission format (slide/page limits, template)                    | Not verified                                                         | Check the portal template; this document is the backing plan and a short submission version is needed |
| Official evaluation/scoring granularity (pixel / tile / pair)               | Not stated in the brief                                              | Watch for organiser clarifications                                                                    |
| Organiser data format and test-set structure                                | Not stated beyond "common geospatial formats such as GeoTIFF or COG" | Watch for organiser clarifications                                                                    |
| Organiser hardware constraints                                              | Not stated                                                           | Ask or watch for clarifications                                                                       |
| OSCD: 24 Sentinel-2 pairs, 2015–2018, urban-focused binary labels           | Confirmed from dataset pages                                         | Confirm the licence for the **change labels** and whether all pair labels are public                  |
| Other dataset licences (land cover, DEM, imagery)                           | Not checked                                                          | Check before packaging                                                                                |
| Retrieval model licences and weight availability                            | Not checked                                                          | Check per candidate                                                                                   |
| Sentinel-2 processing-baseline reflectance offset (early 2022)              | From general knowledge, not checked against ESA documentation        | Verify in ESA product documentation during the spike                                                  |
| Vector-index behaviour (deletion, training needs) for the chosen index type | From general knowledge                                               | Test on the chosen index type                                                                         |

### B. Must measure experimentally

- retrieval model (against R0) and query robustness
- tile size
- candidate rate and the value of the union
- registration effect (conditional correction vs none)
- processing-baseline correction (unit test)
- seasonal false alarms and same-season pairing effect
- temporal false narrowing
- calibration method (C0 / C1 / C2)
- operating threshold and achievable target
- MMU and τ
- clustering quality (unsupervised)
- storage, runtime, memory, ingest time

### C. Must freeze before final evaluation

- model versions and weights
- preprocessing version, including baseline handling
- registration threshold
- calibration method
- precision-target protocol and threshold
- MMU, τ and aggregation rule
- temporal search configuration and whether narrowing is enabled
- dataset split
- random seeds

Recorded in `evaluation_config.yaml`.

### D. Must not claim without evidence

- universal retrieval superiority
- an exact change date
- guaranteed seasonal safety of temporal narrowing
- calibrated change-type probabilities
- generalisation beyond the tested geography
- a "False-Alarm Firewall" improvement without an ablation showing it
- precise confidence with too few independent scenes
- clustering accuracy without ground truth

### Feasibility numbers in section 9

The minimum-event table is computed from the Wilson lower-bound formula for all-correct events at 95% confidence. It is a necessary-condition check only, and it is recomputed if a different confidence level is used.
