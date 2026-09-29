# Problem Stament:Semantic Retrieval and Multi-Temporal Change Analysis of Satellite lmagery.

> Main Topics and Percentage:

> RAG -> 15%

> Geospatial ingestion and preprocessing -> 25%

> Change detection and classification ->25%

> False-alarm suppression ->15%

> Discovery and clustering ->5%

> Analyst workflow and audit trail ->10%

> Infrastructure and evaluation ->10%

## Meaning:

- <b>Temporal</b>:Relating to time. In satellite imagery, it refers to when an image was captured. A single image is a snapshot of one moment eg.a city on 12 March 2024

- <b>Multi-Temporal</b>:Using images of the same location captured at different times, such as 2015, 2020 and 2025. This lets you compare how the area changes over time

- <b>Spectral</b>:Relating to wavelength. It refers to which part of the electromagnetic spectrum a sensor records (visible light, near-infrared, shortwave infrared, thermal, and so on). Different surfaces reflect different wavelengths: healthy vegetation reflects strongly in near-infrared, while water absorbs it. That is why spectral information helps tell land-cover types apart (indices such as NDVI and NDWI are built on it).

- <b>Multi-Spectral</b>:Imagery captured in several separate wavelength bands at the same time, typically 3 to 15 bands, such as Blue, Green, Red, NIR and SWIR. Sentinel-2 (13 bands) and Landsat are common examples. It gives much more information than an ordinary RGB photo, which improves classification and lets you detect things invisible to the naked eye, like vegetation stress. (Hyperspectral goes further, with hundreds of narrow bands.)

- <b>Multi-Sensor Imagery</b>:Combining data from different satellites or sensor types of the same area, for example optical imagery (Sentinel-2), radar/SAR (Sentinel-1), and higher-resolution commercial imagery.

- <b>Free-text search over imagery tiles using natural-language queries</b>:That phrase describes semantic (text-to-image) retrieval: you type a plain-English query like "flooded farmland near a river" and the system returns the image tiles that match, without needing keywords, labels, or metadata tags.

## 1.Overview:

<p>
Semantic Retrieval and Multi-Temporal Change Analysis of Satellite Imagery is a problem statement (SIH26227) from the Smart India Hackathon (SIH) 2026, posed by the Ministry of Defence (MoD). The challenge asks teams to build a system that makes large satellite-imagery archives queryable by semantic content (natural language or example images) and by change over time, while still supporting conventional spatial, temporal, and sensor fil
</p>

<b>Description:</b>

<p>
<b>Background</b>:
Earth-observation archives are expanding rapidly and increasingly contain multi-temporal, multi-spectral and multi-sensor imagery.

<mark>Conventional catalogues are effective for searching by metadata such as coordinates, acquisition date, platform and product type, but analysts may still need to know where and when to look before they can examine the imagery itself</mark>.

Recent advances in multimodal and remote-sensing foundation models have improved semantic representation of Earth-observation data, creating the possibility of searching imagery by meaning as well as metadata.<mark>Translating those advances into a reliable operational system remains challenging</mark>. particularly when the<mark>system must work on-premises, ingest new acquisitions incrementally, preserve geospatial provenance, and suppress false change caused by season, atmosphere, viewing geometry, registration error or sensor differences<mark>.

<b>Detailed Description</b>:
Teams are required to build a system that makes a<mark>satellite-imagery archive queryable by semantic content and by change over time, while retaining conventional spatial, temporal and sensor filters</mark>.

The solution should<mark>support analyst discovery rather than require the analyst to identify every location of interest in advance</mark>.

<b>Six core capabilities are required</b>:

<ol>
<li><b>Semantic and Multimodal Retrieval</b>:Support <mark>free-text search over imagery tiles</mark>using natural-language queries, together with <mark>image-to-image search</mark> for visually and semantically similar locations.<mark>Results should be rank ordered</mark> and may be refined using <mark>area-of-interest, date-range, sensor or other metadata filters</mark>. Example queries include "newly built structures near a river" and "large vehicle concentrations on open ground".</li>

<li><b>Multi-Temporal Change Analysis</b>: For a <mark>specified area and time window, identify meaningful changes such as appearance, disappearance, expansion or contraction of features</mark>;<mark>classify</mark> supported change types such as <mark>construction, clearance, water-extent variation or road development</mark>; and estimate the earliest available observation at which the change is supported by usable imagery.</li>

<li><b>False-Alarm Suppression and Quality Handling</b>: Seasonal variation, illumination and view-angle differences, cloud, haze, snow, shadows, radiometric inconsistency and imperfect co-registration must be treated as confounding factors rather than automatically reported as change. The system should <mark>use quality masks, normalization, confidence estimates or equivalent mechanisms and should favour analytically useful precision over indiscriminate change recall</mark>.</li>

<li><b>Discovery and Clustering</b>: Support <mark>unsupervised or embedding-based grouping of similar sites across a wider area</mark> so that an analyst who identifies one location of interest can discover other locations with comparable visual or semantic characteristics without manually constructing a new query for each site.</li>

<li><b>Analyst Workflow and Provenance</b>: Provide a <mark>ranked review queue with before-and-after evidence, location, acquisition time, sensor or source information, confidence and relevant processing history</mark>. Analysts should be able to <mark>confirm or reject candidates, preserve those decisions in the audit trail, and use feedback for subsequent reranking or refinement where the chosen approach supports it</mark>. Exported results must retain source-scene and processing provenance.

<li><b>Scale, Incremental Ingestion and Sovereignty</b>: Support <mark>efficient vector or equivalent indexing, incremental addition of newly acquired imagery without a complete index rebuild, and complete on-premises operation without cloud services or external APIs during evaluation</mark>. Georeferencing and acquisition metadata must be preserved, and the solution should <mark>ingest organiser-defined common geospatial formats such as GeoTIFF or Cloud Optimized GeoTIFF (COG).</mark>
</ol>
<b>Constraints</b>:

-> The complete demonstration must run with network access.

->disabled after all approved models, libraries and datasets have been staged.

->locally.

->Pretrained public models may be used provided that their origin and licence are declared and the required weights are packaged for offline use.

-><mark>The evaluation</mark> will use publicly available or organiser-generated imagery only; no classified, operational or service-generated imagery will be included.

<b>Expected Solution</b>:
A working system will be demonstrated over an organiser-defined area of interest and time span using public imagery.

Retrieval will be evaluated against held-out semantic queries and relevance judgements, while change analysis will be evaluated against a held-out set of labelled change and no-change cases that participating teams have not seen.

Teams must submit source code, an architecture note, the index-build and incremental-ingestion procedure, model and dataset provenance, and a reproducible evaluation report stating the indexed area, number of scenes or tiles, build time, storage footprint, query latency and hardware used.

</p>

<b>Datasets</b>

<p>
All datasets must be publicly accessible under applicable licences or provided by the organisers. No classified, operational or service-generated data will be used.

-> <b>Primary Imagery Sources.</b>

<ul>
<li>Copernicus Sentinel-2 optical imagery and Sentinel-1 SAR</li>
<li>USGS Landsat Collection 2
<li>NRSC/ISRO Bhuvan open Earth-observation data product</li>
</p>
