# EOR Screening Tool — Open Source

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Version](https://img.shields.io/badge/version-5.0-green.svg)]()

An open-source, collaborative EOR (Enhanced Oil Recovery) screening tool built from reservoir engineering best practices. Screen up to 12 EOR methods across 12 reservoir parameters with transparent, editable scoring logic.

**Live demo:** [your-org.github.io/eor-screening-tool](https://your-org.github.io/eor-screening-tool)

---

## Features

- **12 EOR methods** — Gas injection (CO₂ miscible/immiscible, HC gas, WAG, N₂), Chemical (Polymer, ASP, SP), Thermal (Steam, ISC, Hot Water), Low-Sal WF
- **12 reservoir parameters** — API, viscosity, porosity, oil saturation, depth, temperature, permeability, net pay, Dykstra-Parsons, TAN, TDS, divalents
- **Transparent scoring** — triangular scoring function, fully documented, editable Min/Ideal/Max ranges and weights (0–5)
- **Logistics checklist** — Yes/Partial/No per method with soft-cap logic
- **Tuning panel** — adjust Technical/Logistics balance, MMP caps, rock-type modifiers, water treatment thresholds, Strategic Fit weights
- **Strategic Fit** — independent value-based ranking (Potential × Cost × Speed)
- **Export** — JSON (full state) and CSV (results table)
- **Zero dependencies** — pure HTML/CSS/JS, runs in any browser, no build step needed

---

## Quickstart

```bash
# Clone and open in browser — no npm, no build step
git clone https://github.com/your-org/eor-screening-tool.git
cd eor-screening-tool
open index.html       # macOS
xdg-open index.html   # Linux
start index.html      # Windows
```

Or deploy to GitHub Pages:
1. Fork this repo
2. Go to Settings → Pages → Deploy from branch `main` / `root`
3. Your tool will be live at `https://your-username.github.io/eor-screening-tool`

---

## Repository Structure

```
eor-screening-tool/
├── index.html            # Complete self-contained application (HTML + CSS + JS)
├── src/
│   └── scoringEngine.js  # Scoring engine as ES module (importable / Node.js compatible)
├── data/
│   └── criteria.json     # Screening criteria as JSON (for Google Sheets integration)
├── tests/
│   └── scoring.test.js   # Unit tests for scoring functions
└── README.md
```

---

## Scoring Logic

### Triangular Scoring Function

Each reservoir parameter is scored 0–100 against [Min, Ideal, Max] ranges:

```
value < Min          → score = 0        (below range)
Min ≤ value ≤ Ideal  → score = (value − Min) / (Ideal − Min) × 100
Ideal ≤ value ≤ Max  → score = (Max − value) / (Max − Ideal) × 100
value > Max          → score = 0        (above range)
Max = 999            → no upper limit   (score stays 100 once value ≥ Ideal)
```

### Weighted Technical Score

```
TechScore = Σ(param_score × weight) / Σ(weight × 100) × 100
```

Only parameters with `weight > 0` and non-empty field values are included.

### Combined Score

```
Combined = (Technical × techWeight% + Logistics × logWeight%) / 100
```

Default: 65% Technical + 35% Logistics.

### Soft-Caps

- **MMP flag** (gas methods): if current reservoir pressure < MMP, Technical is capped at `techSoftCap` (default 30%)
- **Logistics soft-cap**: if a critical logistics question = "No", Logistics is capped at `logSoftCap` (default 30%)
- **Fractured carbonate**: Technical hard-capped at `fracturedCarbCap` (default 30%) for all methods except Low-Sal WF

### Weights (0–5)

| Weight | Meaning |
|--------|---------|
| 0 | Parameter not used for this method |
| 1 | Minor influence |
| 2 | Moderate influence |
| 3 | Significant influence |
| 4 | High importance |
| 5 | Critical — dominates score |

---

## Modifying the Criteria

### Option 1: In-App (no code)

Use the **④ Screening Criteria & Weights** tab to edit Min/Ideal/Max/Weight per parameter and method. Changes apply on the next screening run.

### Option 2: Edit `scoringEngine.js`

```javascript
// src/scoringEngine.js — SCREENING_CRITERIA object
"Polymer Flood": {
  temp: { min: 20, avg: 60, max: 120, weight: 3 },   // ← edit these
  tds:  { min: 0,  avg: 30000, max: 250000, weight: 4 },
  // ...
}
```

### Option 3: Link Google Sheets (collaborative)

Export `data/criteria.json` to a Google Sheet with the Sheets API:

```javascript
// Example: load criteria from published Google Sheet
const SHEET_URL = "https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/gviz/tq?tqx=out:json";
const resp = await fetch(SHEET_URL);
const criteria = parseSheetsResponse(resp);
```

See `docs/google-sheets-integration.md` for the full guide (column mapping, JSON schema, auto-refresh).

---

## Contributing

We welcome contributions from reservoir engineers, data scientists, and developers.

### How to Contribute

1. **Fork** this repository
2. **Create a branch**: `git checkout -b feature/polymer-salinity-update`
3. **Edit** `src/scoringEngine.js` or `index.html`
4. **Test**: open `index.html` and verify your changes
5. **Submit a Pull Request** with a description of what you changed and why

### Contribution Ideas

- [ ] Update screening ranges for a specific method based on new field data
- [ ] Add a new EOR method (e.g., thermal/chemical hybrid)
- [ ] Improve Strategic Fit scoring with economic inputs (CAPEX/OPEX tiers)
- [ ] Add uncertainty ranges / Monte Carlo mode for input parameters
- [ ] Multi-field comparison view (compare 5+ fields side-by-side)
- [ ] Import from CSV / Excel
- [ ] Google Sheets live integration

### Changing a Weight or Range

When you update a weight or range, please include in your PR:
- The scientific rationale or field data supporting the change
- The literature reference (SPE paper, textbook, field case)
- Before/after comparison using at least one demo field

---

## References

| Source | Coverage |
|--------|----------|
| Taber, Martin & Seright (1997) — *SPE-35385* | Original screening criteria tables |
| Sheng, J.J. (2014) — *Enhanced Oil Recovery Field Case Studies* | Method-specific ranges |
| Al Adasani & Bai (2011) — *JPT* | Statistical database analysis of 100+ EOR projects |
| Hartono et al. (2017) | Chemical EOR screening framework |
| Seright et al. (2021–2023) | Polymer flooding recommendations |
| Delamaide (2018) | Polymer flood field performance |
| Dupuis et al.; Thomas & Seright (2026) | Polymer bank size and field data |

> **Important limitation:** This tool provides indicative screening only. Validate results with detailed reservoir engineering, laboratory corefloods, and full economic analysis before pilot or field decisions. MMP should be measured by slim-tube test or estimated via Cronquist/Glaso/Yelin correlations.

---

## License

MIT License — free to use, modify, and distribute. See [LICENSE](LICENSE).

---

## Authors

- Antoine Thomas (eppok) — original Excel model, reservoir engineering methodology
- Contributors welcome — see [Contributing](#contributing)
