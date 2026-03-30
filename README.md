# DAPI + Lipid Droplet Analyzer

A browser-based dual-channel fluorescence image analysis tool for counting DAPI-stained nuclei and measuring green-channel (lipid) fluorescence — including vesicle pattern detection.

**No installation required. Works entirely offline in the browser.**

---

## Features

- **DAPI channel**: nucleus detection using adaptive thresholding + connected components
- **Green channel**: fixed-radius measurement zone per cell
- **Overlap exclusion**: cells with overlapping measurement zones are excluded automatically
- **Vesicle analysis**: classifies each cell as *vesicular / mixed / diffuse* using Coefficient of Variation (CV)
- **Signal quantification**: integrated density, mean intensity, signal per area unit
- **Copyable summary report** with histogram

---

## Usage

1. Open `index.html` in **Firefox**, or serve locally:
   ```bash
   python -m http.server 8080
   ```
   Then visit `http://localhost:8080`

2. Drop your **DAPI image** into the left zone and your **green (lipid) image** into the right zone

3. Adjust parameters as needed and click **▶ נתח**

> **Note:** Opening directly via `file://` in Chrome may block file reading. Use Firefox or a local server.

---

## Parameters

| Parameter | Description |
|---|---|
| Adaptive sensitivity | Threshold offset for DAPI nucleus detection |
| Min / Max radius | Filter detected nuclei by size (px) |
| Circularity | Minimum roundness score (0–1) |
| Green measurement radius | Fixed radius (px) around each nucleus centroid for green channel sampling |
| Green intensity threshold | Pixel intensity cutoff for "positive" green signal |

---

## Vesicle Classification

Each valid cell is classified based on the **Coefficient of Variation** (CV = σ/μ) of pixel intensities within the measurement zone:

| CV | Pattern |
|---|---|
| < 0.30 | Diffuse (מפוזר) |
| 0.30 – 0.60 | Mixed (מעורב) |
| > 0.60 | Vesicular (וסיקולרי) |

Local hotspots (connected regions > mean + 1.5σ) are counted separately as **vesicle foci**.

---

## Output

- Per-cell table: radius, DAPI intensity, green mean, integrated density, signal/px, CV, foci count, pattern
- Summary panel with copyable text report
- Size distribution histogram
- Interactive overlay on image (hover for details, click to select)

---

## Technical Notes

- Pure vanilla JavaScript + HTML5 Canvas — no dependencies
- Adaptive threshold via integral image (O(N))
- Box blur approximation of Gaussian (3 passes)
- Morphological open (erode + dilate) for noise removal
- Iterative BFS connected components (no recursion → no stack overflow)
- Touching nucleus separation via IQR median radius heuristic

---

## License

MIT
