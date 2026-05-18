# 🛣️ TCVN 8867:2011 Digital Toolkit
### Benkelman Beam Pavement Analyser

**Live App → [hengkimleng.github.io/TCVN-8867-2011-Digital-Toolkit](https://hengkimleng.github.io/TCVN-8867-2011-Digital-Toolkit/)**

> Developed by **D.Eng. Heng Kim Leng, P.E.** | Deputy Director General, GDPW & GDSM, MPWT Cambodia  
> ATZ+ Partners · BEC Reg. No. 002494 · ASEAN Engineer AE 13045

---

## Overview

This web application digitises the full calculation workflow of **TCVN 8867:2011** (*Flexible Pavement — Determination of Elastic Modulus Using Benkelman Beam*), transforming raw field deflection data into structured, GIS-ready pavement performance intelligence.

The tool is designed for **Cambodia's Ministry of Public Works and Transport (MPWT)** and regional **CLMV** highway asset management programmes. It operates entirely in the browser — no installation, no server, no internet connection required after the page loads.

---

## Key Features

### ✅ Full TCVN 8867:2011 Compliance
| Section | Implementation |
|---------|----------------|
| §6.1.1 | Corrected deflection: `L.itt = (1/Kq) · Km · Kt · Li` |
| §6.2.3 | Characteristic deflection: `L.seg = L.avg + K·δ` |
| §6.2.4 | Resilient modulus: `MR = 0.71·p·D·(1−μ²) / L.seg` |
| Annex E | Statistical outlier exclusion (Lk statistic, α = 0.05 or 0.10) |
| Annex F | Cumulative Difference Sums (CDS/Zx) automatic segmentation |

### 📐 Multi-Standard MR Thresholds
The Good / Fair / Weak rating thresholds adapt automatically to the selected road class and design standard:

| Standard | Class | K | MR Good | MR Min (Pass) |
|----------|-------|---|---------|---------------|
| **TCVN / 22TCN** | Expressway / Class I | 2.0 | ≥ 300 MPa | ≥ 200 MPa |
| | Class II / Urban Arterial | 1.64 | ≥ 200 MPa | ≥ 140 MPa |
| | Class III | 1.3 | ≥ 140 MPa | ≥ 100 MPa |
| | Class IV–VI / Local | 1.04 | ≥ 100 MPa | ≥ 70 MPa |
| **JTG D50** (China) | Grade 1 (一级) | 2.0 | ≥ 250 MPa | ≥ 180 MPa |
| | Grade 2 (二级) | 1.64 | ≥ 180 MPa | ≥ 117 MPa |
| | Grade 3 (三级) | 1.3 | ≥ 117 MPa | ≥ 80 MPa |
| | Grade 4 (四级) | 1.04 | ≥ 80 MPa | ≥ 50 MPa |
| **AASHTO** | Principal Arterial | 2.0 | ≥ 300 MPa | ≥ 200 MPa |
| | Minor Arterial | 1.64 | ≥ 200 MPa | ≥ 150 MPa |
| | Collector | 1.3 | ≥ 150 MPa | ≥ 100 MPa |
| | Local Road | 1.04 | ≥ 100 MPa | ≥ 70 MPa |

### 📊 Interactive Dashboard (5 Tabs)
- **Overview** — Summary stats, deflection histogram, MR distribution bar chart, CDS segmentation plot
- **Deflection Profile** — Full longitudinal profile with raw Li, corrected L.itt, outliers flagged, and L.seg design line overlay
- **Segments & MR** — Per-segment table with L.avg, δ, L.seg, MR (MPa), and Good/Fair/Weak rating
- **Raw Data** — Point-by-point corrected values with outlier flags and segment assignment
- **LRS Event Table** — ISO 19148 Linear Referencing System output, ready for ArcGIS / QGIS import

### 🗂️ GIS & LRS-Ready ETL Pipeline
- Input CSV is parsed with smart column matching (`Route_ID`, `Point_ID`, `Chainage`, `Deflection_Li_mm`)
- Chainage accepts `Km 46+000`, `46+000`, or plain metre values
- LRS CSV export: `Route_ID, From_Measure, To_Measure, MR_MPa` — droppable directly into ArcGIS or QGIS as a route event layer
- Output filenames automatically embed the source survey filename (e.g. `LRS_Benkelman_NR10_2026.csv`)

### 🔒 Offline-First & Resilient
- Inline fallback CSV parser — works even if CDN is blocked
- BOM (`\uFEFF`) stripping for Excel-exported UTF-8 files
- Detailed error toasts for malformed files, empty datasets, or missing columns
- Downloadable CSV template for field teams

---

## Input CSV Format

```csv
Route_ID,Point_ID,Chainage,Deflection_Li_mm
NR10,1,Km 0+000,0.4636
NR10,2,Km 1+000,0.4462
NR10,3,Km 2+000,0.4734
```

| Column | Format | Notes |
|--------|--------|-------|
| `Route_ID` | Text | e.g. `NR10`, `NR5`, `NR60B` |
| `Point_ID` | Integer | Sequential measurement point number |
| `Chainage` | `Km 46+000` or `46000` | Both formats parsed automatically |
| `Deflection_Li_mm` | Decimal (mm) | Raw Benkelman reading in millimetres |

> **Unit note for Chinese QC reports:** Values reported in `0.01 mm` units must be divided by 100 before loading. Example: representative value `46.36` → enter as `0.4636` mm.

---

## Included Sample Files

| File | Description |
|------|-------------|
| `NR10_Deflection_2026.csv` | 189 sections, Km 0+000 → Km 196+000 (2026 survey) |
| `NR10_Deflection_2025.csv` | 189 sections, Km 0+000 → Km 196+000 (2025 survey) |
| `NR10_150km_BenkelmanSurvey.csv` | 1,500 synthetic points, 10 pts/km, zoned deflection model |

---

## Corrections Applied (TCVN 8867:2011)

```
L.itt = (1 / Kq) × Km × Kt × Li
```

| Factor | Parameter | Typical Value |
|--------|-----------|---------------|
| `Kq` | Load correction (axle load vs standard) | 1.0 if standard 100 kN axle |
| `Km` | Season correction (worst-case moisture) | 1.0–1.47 per Annex D, Table D.2 |
| `Kt` | Temperature correction | Per Annex D, formula D.1 |

---

## Segmentation Method (Annex F)

The app implements the **Cumulative Difference Sums (CDS)** method:

1. Compute `Zx = ΣSi − Φ · Σ∆xi` for each measurement point
2. Identify segment boundaries at sign changes in the slope of the Zx curve
3. Merge segments shorter than the minimum length (default 500 m) with adjacent sections
4. Apply Annex E outlier test within each segment before computing L.avg and δ

---

## Export Outputs

### LRS CSV (`LRS_Benkelman_<filename>.csv`)
```csv
Route_ID,From_Measure,To_Measure,Point_Count,Mean_Deflection_Lavg_mm,Char_Deflection_Lseg_mm,MR_MPa
NR10,0.000,7000.000,7,0.4623,0.5891,128.3
```

### Analysis Report (`Benkelman_Report_<filename>.txt`)
```
BENKELMAN BEAM ANALYSIS REPORT
TCVN 8867:2011
============================================================

Survey File  : NR10_Deflection_2026.csv
Route        : NR10
Generated    : 18/05/2026, 14:32:05 (ICT)
Sections     : 12 homogeneous segments
Measurement  : 189 points  (2 outliers removed)

PARAMETERS
Road Class   : Grade 2 (二级) — JTG D50 China (K=1.64)
MR Thresholds: Good ≥ 180 MPa | Fair ≥ 117 MPa | Weak < 117 MPa
...
```

---

## Technical Architecture

- **Single-file HTML** — zero dependencies to install; all logic, styles, and fallback parser are self-contained
- **Chart.js 4.4** — deflection profile, histogram, MR bar chart, CDS segmentation plot
- **PapaParse 5.4** — primary CSV parser (CDN); inline fallback parser activates automatically if CDN is unavailable
- **Inline CSV fallback** — handles quoted fields, CRLF line endings, and BOM characters
- **ISO 19148 LRS** — output measures in metres, compatible with ArcGIS Linear Referencing and QGIS Route Events

---

## Roadmap

- [ ] Khmer / English UI language toggle
- [ ] FWD (Falling Weight Deflectometer) module
- [ ] IRI overlay from companion survey data
- [ ] Multi-route batch processing
- [ ] PDF report export with MPWT letterhead
- [ ] Integration with GIAMS (General Infrastructure Asset Management System)

---

## References

- TCVN 8867:2011 — *Áo đường mềm – Xác định mô đun đàn hồi chung của kết cấu bằng cần đo võng Benkelman*
- 22 TCN 211-06 — *Áo đường mềm – Các yêu cầu và chỉ dẫn thiết kế*
- JTG D50-2017 — *Highway Asphalt Pavement Design Specification* (Ministry of Transport, China)
- AASHTO Guide for Design of Pavement Structures (1993)
- ISO 19148:2021 — *Geographic information — Linear referencing*
- TCVN 4054:2005 — *Đường ô tô – Yêu cầu thiết kế*

---

## Contact

**D.Eng. Heng Kim Leng, P.E.**  
Deputy Director General — GDPW & GDSM, MPWT Cambodia  
Founder — ATZ+ Partners  
📧 hengkimleng@gmail.com  
📞 +855 17 596 168 / 097 299 6565  
🔗 BEC Reg. No. 002494 · ASEAN Engineer AE 13045

---

*© 2026 MPWT Cambodia / ATZ+ Partners. Built for Digital-First infrastructure management under Cambodia's Pentagon Strategy Phase 1.*
