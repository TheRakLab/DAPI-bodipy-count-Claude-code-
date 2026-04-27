[README.md](https://github.com/user-attachments/files/27014509/README.md)
# 🔬 CellScope — Browser-Based Cell Analysis Tool

[![GitHub Pages](https://img.shields.io/badge/Live%20Demo-GitHub%20Pages-brightgreen?style=flat-square&logo=github)](https://YOUR-USERNAME.github.io/cellscope/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![No Install](https://img.shields.io/badge/No%20Install-Runs%20in%20Browser-orange?style=flat-square)]()
[![Zero Dependencies](https://img.shields.io/badge/Backend-None-lightgrey?style=flat-square)]()

A fully client-side fluorescence microscopy analysis tool. Upload DAPI and GFP image pairs, auto-detect nuclei, count differentiated cells manually, and export results — all without leaving your browser. No installation, no server, no data leaves your machine.

---

## ✨ Features

| Feature | Details |
|---|---|
| 🔵 **DAPI Auto-Detection** | Adaptive threshold + morphological operations + connected-component analysis |
| 🟢 **Green Channel Overlay** | Screen-blend composite with per-layer opacity control |
| 🖱️ **Manual Counting** | Click to place nucleus / differentiated-cell markers; right-click to remove |
| 📥 **Import Auto Results** | One-click import of auto-detected nuclei as manual starting point |
| 📊 **Excel Export** | Multi-sheet `.xlsx` with summary, per-nucleus details, and manual markers |
| 📄 **CSV Export** | Single-row summary for batch analysis pipelines |
| 🗂️ **Google Sheets Sync** | Append a row to your lab spreadsheet via Apps Script webhook |
| 💾 **Local History** | Persist all sessions in browser `localStorage` — survives page refresh |
| 🖨️ **Print Summary** | Clean print layout for lab notebooks |

---

## 🚀 Live Demo

👉 **[Open CellScope](https://theraklab.github.io/DAPI-bodipy-count-Claude-code-/)**

No login required. All processing happens in your browser.

---

## 📸 Screenshots

> *(Replace with actual screenshots)*

| Upload | Analysis | Manual Count | Export |
|--------|----------|--------------|--------|
| ![Upload step](docs/screenshot_upload.png) | ![Analysis step](docs/screenshot_analysis.png) | ![Manual count](docs/screenshot_manual.png) | ![Export step](docs/screenshot_export.png) |

---

## 🛠️ How to Use

### Step 1 — Upload Images
- Fill in sample metadata (name, cell type, date, condition, magnification)
- Upload a **DAPI / Hoechst** image (nuclei channel)
- Upload a **GFP / Alexa488** image (differentiation marker)
- Drag-and-drop or click to select

### Step 2 — Auto Analysis
Tune four parameters to optimize nucleus detection:

| Parameter | What it does |
|---|---|
| **Adaptive sensitivity** | Local threshold bias (negative = more permissive) |
| **Min size (px radius)** | Ignore objects smaller than this |
| **Max size (px radius)** | Ignore objects larger than this |
| **Circularity threshold** | 0 = any shape, 1.0 = perfect circle only |

Click **▶ Analyze Nuclei**. Detected nuclei are drawn as colored ellipses.

Use the **Channel Mix** controls to tune DAPI and Green overlay opacity while reviewing results.

### Step 3 — Manual Count
- Switch between **DAPI**, **Green**, and **Overlay** views
- In Overlay mode, use the **Channel Mix** sliders to independently control DAPI and Green brightness
- Click the canvas in **🔵 nucleus mode** to mark nuclei
- Click in **🟢 diff. cell mode** to mark GFP+ differentiated cells
- Right-click any marker to remove it
- Press **Auto 🔵** to import auto-detected nuclei as a starting point

### Step 4 — Export
- **Export to Excel (.xlsx)** — summary sheet + per-nucleus data + manual markers
- **Export CSV** — single row for your pipeline
- **Save to Google Sheets** — append to a running lab spreadsheet
- **Save Locally** — saves to browser storage, visible in the **Sheet 📋** tab
- **Print Summary** — formatted print layout

### Sheet 📋 Tab
View all sessions saved to this browser. Columns include:
`Saved At · Sample Name · Cell Type · Condition · Magnification · DAPI File · Green File · Auto Nuclei · Avg Area · Manual Nuclei · Diff. Cells · Diff. %`

Export the full history as `.xlsx` or `.csv`, or delete individual rows.

---

## 🔗 Google Sheets Integration

CellScope can append one row per analysis to a Google Sheet via a simple Apps Script webhook.

### Setup (one-time, ~3 minutes)

1. Open [Google Sheets](https://sheets.google.com) and create a new spreadsheet
2. Go to **Extensions → Apps Script**
3. Delete the default code and paste the script below
4. Click **Save**, then **Deploy → New Deployment**
5. Choose **Web app**, set **Execute as: Me**, **Who has access: Anyone**
6. Copy the generated URL and paste it into CellScope's Google Sheets modal

### Apps Script

```javascript
function doPost(e) {
  const data = JSON.parse(e.postData.contents);
  const sheet = SpreadsheetApp
    .getActiveSpreadsheet()
    .getActiveSheet();

  if (sheet.getLastRow() === 0) {
    sheet.appendRow([
      'saved_at','sample_name','cell_type',
      'exp_date','condition','magnification',
      'auto_nuclei','avg_area_px2',
      'manual_nuclei','diff_cells','diff_pct'
    ]);
    sheet.getRange(1,1,1,11)
      .setFontWeight('bold')
      .setBackground('#0f9d58')
      .setFontColor('#ffffff');
  }

  sheet.appendRow([
    data.savedAt, data.sampleName, data.cellType,
    data.date, data.condition, data.objective,
    data.autoCount, data.avgArea,
    data.manualNuclei, data.manualDiff, data.diffPercent
  ]);

  return ContentService
    .createTextOutput(JSON.stringify({status:'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

> **Note:** Because CellScope uses `mode: 'no-cors'` to avoid CORS errors, it cannot read the response. A successful send shows ✅ immediately. If your sheet isn't updating, check that the deployment is set to **Anyone** access.

---

## 🌐 Deploying to GitHub Pages

```bash
# 1. Create a new repo on GitHub (e.g. "cellscope")
# 2. Clone it locally
git clone https://github.com/YOUR-USERNAME/cellscope.git
cd cellscope

# 3. Copy the files
cp /path/to/index.html .
cp /path/to/README.md .

# 4. Push
git add .
git commit -m "Initial release"
git push origin main

# 5. Enable GitHub Pages
# Go to: Settings → Pages → Source: Deploy from branch → Branch: main → / (root)
# Your site will be live at: https://YOUR-USERNAME.github.io/cellscope/
```

---

## 🧬 Algorithm Details

Nucleus detection runs entirely in the browser using a custom algorithm:

1. **Grayscale conversion** — luminance-weighted (0.299R + 0.587G + 0.114B)
2. **Box blur** — 2px radius smoothing to reduce noise
3. **Adaptive threshold** — per-pixel local block comparison (block radius ≈ image_size / 20)
4. **Morphological opening** — erosion then dilation to remove small noise
5. **Connected components** — 4-connected flood-fill labeling
6. **Blob fitting** — each blob is fit to an ellipse; centroid, axes, area, and circularity are computed
7. **Filtering** — blobs are discarded if outside the min/max radius range or below the circularity threshold

All steps run in `setTimeout` chains to keep the UI responsive.

---

## 📁 Repository Structure

```
cellscope/
├── index.html       # The entire application (single file, no build step)
└── README.md        # This file
```

The app is intentionally a single HTML file with no external dependencies except:
- [SheetJS (xlsx)](https://sheetjs.com/) — Excel export (loaded from CDN)
- [Google Fonts](https://fonts.google.com/) — IBM Plex Mono + Inter (loaded from CDN)

---

## 🔒 Privacy

- **All image processing happens in your browser.** Images are never uploaded to any server.
- **Local history** is stored in your browser's `localStorage` only — it never leaves your device.
- **Google Sheets sync** sends only the numeric summary (no images) to your own Apps Script endpoint.

---

## 📋 Roadmap / Ideas

- [ ] Brightness / contrast adjustment per channel before analysis
- [ ] Batch mode: process multiple image pairs at once
- [ ] ROI (region of interest) selection
- [ ] Colocalization analysis (DAPI + GFP overlap)
- [ ] Export detected ellipses as ImageJ/FIJI ROI format

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 🙏 Credits

Built with vanilla HTML/CSS/JS. Cell analysis algorithm inspired by classic blob-detection pipelines (watershed-lite, adaptive thresholding).
