### Technical Approach

**Technology Stack**

- **Languages:** Python, SQL, JavaScript/HTML/CSS
- **ML & Processing:** PyTorch, Rasterio/GDAL, remote-sensing vision-language models
- **Search & Database:** FAISS, PostgreSQL/PostGIS or SQLite/SpatiaLite
- **Backend & UI:** FastAPI, Leaflet
- **Data:** Sentinel-2 L2A imagery in GeoTIFF/COG format
- **Hardware:** Multi-core CPU, 16–32 GB RAM, optional NVIDIA GPU, SSD storage
- **Deployment:** On-premises and offline-capable

**System Flow**

### SYSTEM FLOW

**Satellite Imagery**
Sentinel-2 L2A
↓
**Preprocessing & Quality Control**
Cloud/Shadow Masking • Correction • Registration • Tiling
↓
**Semantic Retrieval**
Text / Image Query → Ranked Imagery
↓
**Multi-Temporal Change Detection**
Before–After Comparison • Spectral + Embedding Features
↓
**False-Alarm Suppression**
Seasonality • Clouds • Shadows • Registration • Quality
↓
**Evidence & Confidence**
Evidence Fusion • Calibration • Spatial Coherence
↓
**Temporal Analysis**
Earliest Supported Change Interval
↓
**Analyst Review & Verification**
Before/After Images • Change Mask • Type • Confidence
↓
**Final Output**
**Searchable Imagery + Verified Change Events + Provenance**
