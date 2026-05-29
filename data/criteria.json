/**
 * EOR Screening Tool — Scoring Engine
 * Open-source JavaScript implementation of the scoring logic from EOR_Screening_Tool_v5.xlsx
 *
 * Sources: Taber et al. (1997), Sheng (2014), Al Adasani & Bai (2011),
 *          Hartono et al. (2017), Seright et al. (2021–2023), Delamaide (2018),
 *          Dupuis et al., Thomas et al.
 *
 * License: MIT
 * Repository: https://github.com/your-org/eor-screening-tool
 */

// ─────────────────────────────────────────────────────────────────────────────
// 1. SCREENING CRITERIA
//    For each EOR method: parameter ranges [min, avg/ideal, max] and weights (0–5)
//    Weight 0 = not used for that method
//    Weight 5 = critical parameter
// ─────────────────────────────────────────────────────────────────────────────

export const PARAMETERS = [
  { key: "api",        label: "API Gravity",                    unit: "°API"    },
  { key: "viscosity",  label: "Oil Viscosity (reservoir cond.)", unit: "cP"     },
  { key: "porosity",   label: "Porosity",                       unit: "%"       },
  { key: "so",         label: "Oil Saturation",                 unit: "%"       },
  { key: "depth",      label: "Depth (mid-perf TVDss)",         unit: "m"       },
  { key: "temp",       label: "Reservoir Temperature",          unit: "°C"      },
  { key: "perm",       label: "Horizontal Permeability",        unit: "mD"      },
  { key: "netpay",     label: "Net Pay Thickness",              unit: "m"       },
  { key: "dp",         label: "Dykstra-Parsons Coefficient",    unit: "0–1"     },
  { key: "tan",        label: "TAN (Total Acid Number)",        unit: "mg KOH/g"},
  { key: "tds",        label: "Injection Water TDS",            unit: "mg/L"    },
  { key: "divalents",  label: "Divalent Cations (Ca²⁺+Mg²⁺)",  unit: "mg/L"    },
];

/**
 * Screening criteria matrix.
 * Each entry: { min, avg, max, weight }
 * weight: 0 (unused) → 5 (critical)
 */
export const SCREENING_CRITERIA = {
  "Miscible CO2": {
    api:       { min: 22,    avg: 36,     max: 45,     weight: 4, note: "Higher = lighter oil" },
    viscosity: { min: 0.1,   avg: 1.7,    max: 10,     weight: 4, note: "Reservoir conditions" },
    porosity:  { min: 3,     avg: 15,     max: 37,     weight: 2, note: "Effective porosity" },
    so:        { min: 25,    avg: 53,     max: 89,     weight: 3, note: "At EOR start" },
    depth:     { min: 600,   avg: 1880,   max: 4070,   weight: 4, note: "Mid-perforation TVDss" },
    temp:      { min: 20,    avg: 58,     max: 121,    weight: 2, note: "Average reservoir T" },
    perm:      { min: 1,     avg: 60,     max: 1000,   weight: 2, note: "Horizontal, geometric mean" },
    netpay:    { min: 3,     avg: 15,     max: 999,    weight: 1, note: "Net thickness; 999 = no upper limit" },
    dp:        { min: 0,     avg: 0.5,    max: 0.85,   weight: 3, note: "0=uniform; 0.85=heterogeneous" },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0, note: "Total Acid Number — critical for ASP" },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0, note: "Total Dissolved Solids" },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0, note: "Divalents Ca²⁺+Mg²⁺" },
  },
  "Immiscible CO2": {
    api:       { min: 11,    avg: 22,     max: 35,     weight: 3 },
    viscosity: { min: 0.5,   avg: 65,     max: 600,    weight: 2 },
    porosity:  { min: 17,    avg: 26,     max: 32,     weight: 1 },
    so:        { min: 15,    avg: 63,     max: 78,     weight: 3 },
    depth:     { min: 350,   avg: 1030,   max: 2590,   weight: 2 },
    temp:      { min: 20,    avg: 51,     max: 92,     weight: 1 },
    perm:      { min: 10,    avg: 217,    max: 1000,   weight: 2 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.6,    max: 0.85,   weight: 3 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "Miscible HC Gas": {
    api:       { min: 23,    avg: 40,     max: 45,     weight: 4 },
    viscosity: { min: 0.04,  avg: 1,      max: 3,      weight: 3 },
    porosity:  { min: 4,     avg: 15,     max: 45,     weight: 2 },
    so:        { min: 30,    avg: 77,     max: 98,     weight: 3 },
    depth:     { min: 1220,  avg: 1887,   max: 4845,   weight: 4 },
    temp:      { min: 30,    avg: 95,     max: 165,    weight: 2 },
    perm:      { min: 0.1,   avg: 726,    max: 5000,   weight: 2 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.5,    max: 0.85,   weight: 2 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "WAG": {
    api:       { min: 25,    avg: 42,     max: 45,     weight: 3 },
    viscosity: { min: 0.07,  avg: 0.2,    max: 1,      weight: 3 },
    porosity:  { min: 11,    avg: 18,     max: 24,     weight: 2 },
    so:        { min: 40,    avg: 75,     max: 80,     weight: 3 },
    depth:     { min: 1830,  avg: 2775,   max: 5640,   weight: 3 },
    temp:      { min: 80,    avg: 110,    max: 123,    weight: 1 },
    perm:      { min: 50,    avg: 143,    max: 1000,   weight: 2 },
    netpay:    { min: 5,     avg: 15,     max: 999,    weight: 2 },
    dp:        { min: 0,     avg: 0.5,    max: 0.85,   weight: 2 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "N2 Injection": {
    api:       { min: 35,    avg: 40,     max: 45,     weight: 5 },
    viscosity: { min: 0.07,  avg: 0.2,    max: 0.4,    weight: 4 },
    porosity:  { min: 7,     avg: 11,     max: 14,     weight: 1 },
    so:        { min: 40,    avg: 76,     max: 80,     weight: 3 },
    depth:     { min: 1830,  avg: 3754,   max: 5640,   weight: 4 },
    temp:      { min: 70,    avg: 130,    max: 163,    weight: 2 },
    perm:      { min: 0.2,   avg: 15,     max: 35,     weight: 1 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.5,    max: 0.85,   weight: 1 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "Polymer Flood": {
    api:       { min: 13,    avg: 33,     max: 45,     weight: 1, note: "Broad range; viscosity more critical" },
    viscosity: { min: 0.4,   avg: 500,    max: 15000,  weight: 3 },
    porosity:  { min: 10,    avg: 23,     max: 33,     weight: 3 },
    so:        { min: 35,    avg: 72,     max: 92,     weight: 3 },
    depth:     { min: 213,   avg: 2015,   max: 2926,   weight: 1 },
    temp:      { min: 20,    avg: 60,     max: 120,    weight: 3, note: "HPAM→ATBS chemistry adapts below 120°C" },
    perm:      { min: 2,     avg: 817,    max: 15000,  weight: 3 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.6,    max: 0.95,   weight: 3 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 30000,  max: 250000, weight: 4, note: "Polymer chemistry adapts to high TDS" },
    divalents: { min: 0,     avg: 1000,   max: 25000,  weight: 5, note: "MOST important for polymer/chemicals" },
  },
  "ASP Flood": {
    api:       { min: 20,    avg: 33,     max: 45,     weight: 3 },
    viscosity: { min: 1,     avg: 50,     max: 500,    weight: 3 },
    porosity:  { min: 15,    avg: 27,     max: 35,     weight: 2 },
    so:        { min: 35,    avg: 63,     max: 75,     weight: 3 },
    depth:     { min: 152,   avg: 950,    max: 2743,   weight: 2 },
    temp:      { min: 20,    avg: 49,     max: 93,     weight: 5, note: "Critical: alkali/surfactant sensitive to T" },
    perm:      { min: 10,    avg: 487,    max: 1520,   weight: 3 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.6,    max: 0.95,   weight: 2 },
    tan:       { min: 0.2,   avg: 1,      max: 5,      weight: 5, note: "Critical: drives IFT reduction mechanism" },
    tds:       { min: 0,     avg: 20000,  max: 80000,  weight: 5, note: "High TDS kills surfactant performance" },
    divalents: { min: 0,     avg: 500,    max: 5000,   weight: 5, note: "Divalents precipitate surfactant/alkali" },
  },
  "Surfactant-Polymer": {
    api:       { min: 20,    avg: 33,     max: 45,     weight: 3 },
    viscosity: { min: 1,     avg: 11,     max: 35,     weight: 3 },
    porosity:  { min: 15,    avg: 16,     max: 20,     weight: 2 },
    so:        { min: 20,    avg: 48,     max: 53,     weight: 3 },
    depth:     { min: 190,   avg: 943,    max: 2743,   weight: 2 },
    temp:      { min: 25,    avg: 59,     max: 71,     weight: 5 },
    perm:      { min: 10,    avg: 55,     max: 500,    weight: 3 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.6,    max: 0.95,   weight: 2 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 15000,  max: 50000,  weight: 3 },
    divalents: { min: 0,     avg: 500,    max: 5000,   weight: 3 },
  },
  "Steam Flood": {
    api:       { min: 8,     avg: 14,     max: 25,     weight: 5 },
    viscosity: { min: 10,    avg: 1000,   max: 200000, weight: 4 },
    porosity:  { min: 25,    avg: 32,     max: 65,     weight: 3 },
    so:        { min: 40,    avg: 66,     max: 90,     weight: 3 },
    depth:     { min: 60,    avg: 479,    max: 1370,   weight: 5, note: "Depth critical: heat losses increase with depth" },
    temp:      { min: 10,    avg: 79,     max: 177,    weight: 1 },
    perm:      { min: 200,   avg: 2573,   max: 15000,  weight: 4 },
    netpay:    { min: 6,     avg: 20,     max: 999,    weight: 3 },
    dp:        { min: 0,     avg: 0.7,    max: 0.99,   weight: 1 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "In-Situ Combustion": {
    api:       { min: 10,    avg: 20,     max: 40,     weight: 3 },
    viscosity: { min: 5,     avg: 200,    max: 5000,   weight: 3 },
    porosity:  { min: 14,    avg: 23,     max: 35,     weight: 2 },
    so:        { min: 50,    avg: 70,     max: 94,     weight: 3 },
    depth:     { min: 120,   avg: 1698,   max: 3505,   weight: 3 },
    temp:      { min: 15,    avg: 67,     max: 110,    weight: 1 },
    perm:      { min: 10,    avg: 1033,   max: 15000,  weight: 3 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 2 },
    dp:        { min: 0,     avg: 0.6,    max: 0.85,   weight: 2 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "Hot Water Flood": {
    api:       { min: 8,     avg: 15,     max: 30,     weight: 3 },
    viscosity: { min: 10,    avg: 3351,   max: 200000, weight: 3 },
    porosity:  { min: 25,    avg: 31,     max: 37,     weight: 2 },
    so:        { min: 15,    avg: 62,     max: 85,     weight: 3 },
    depth:     { min: 152,   avg: 524,    max: 1370,   weight: 3 },
    temp:      { min: 10,    avg: 37,     max: 138,    weight: 1 },
    perm:      { min: 200,   avg: 2943,   max: 6000,   weight: 3 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.7,    max: 0.99,   weight: 1 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 50000,  max: 300000, weight: 0 },
    divalents: { min: 0,     avg: 5000,   max: 50000,  weight: 0 },
  },
  "Low-Sal. WF": {
    api:       { min: 10,    avg: 30,     max: 45,     weight: 2 },
    viscosity: { min: 0.2,   avg: 5,      max: 2000,   weight: 2 },
    porosity:  { min: 10,    avg: 20,     max: 35,     weight: 2 },
    so:        { min: 30,    avg: 55,     max: 90,     weight: 3 },
    depth:     { min: 100,   avg: 1500,   max: 5000,   weight: 1 },
    temp:      { min: 20,    avg: 60,     max: 150,    weight: 2 },
    perm:      { min: 1,     avg: 100,    max: 5000,   weight: 2 },
    netpay:    { min: 3,     avg: 10,     max: 999,    weight: 1 },
    dp:        { min: 0,     avg: 0.7,    max: 0.99,   weight: 1 },
    tan:       { min: 0,     avg: 1,      max: 5,      weight: 0 },
    tds:       { min: 0,     avg: 5000,   max: 40000,  weight: 2, note: "Needs low-salinity injection water" },
    divalents: { min: 0,     avg: 500,    max: 5000,   weight: 2 },
  },
};

export const EOR_METHODS = Object.keys(SCREENING_CRITERIA);

export const METHOD_CATEGORIES = {
  "Miscible CO2":        "Gas Injection",
  "Immiscible CO2":      "Gas Injection",
  "Miscible HC Gas":     "Gas Injection",
  "WAG":                 "Gas Injection",
  "N2 Injection":        "Gas Injection",
  "Polymer Flood":       "Chemical EOR",
  "ASP Flood":           "Chemical EOR",
  "Surfactant-Polymer":  "Chemical EOR",
  "Steam Flood":         "Thermal EOR",
  "In-Situ Combustion":  "Thermal EOR",
  "Hot Water Flood":     "Thermal EOR",
  "Low-Sal. WF":         "IOR/EOR",
};

// ─────────────────────────────────────────────────────────────────────────────
// 2. TUNING PANEL DEFAULTS
//    All global thresholds and modifiers — editable by user
// ─────────────────────────────────────────────────────────────────────────────

export const DEFAULT_TUNING = {
  // Score balance
  techWeight:          65,   // % of Combined score from Technical
  logWeight:           35,   // % of Combined score from Logistics
  passThreshold:       60,   // Combined ≥ this → candidate
  techSoftCap:         30,   // MMP failure → cap Technical at this %
  logSoftCap:          30,   // Critical logistics "No" → cap Logistics at this %
  fracturedCarbCap:    30,   // Fractured carbonate hard cap (LSWF exempt)

  // Rock-type modifiers (carbonate, non-fractured) — multiplier on Tech score
  // 1.0 = no penalty; 0.5 = 50% penalty
  rockModifiers: {
    "Miscible CO2":        0.5,
    "Immiscible CO2":      0.5,
    "Miscible HC Gas":     0.5,
    "WAG":                 0.5,
    "N2 Injection":        0.5,
    "Polymer Flood":       0.5,
    "ASP Flood":           0.5,
    "Surfactant-Polymer":  0.5,
    "Steam Flood":         0.5,
    "In-Situ Combustion":  0.5,
    "Hot Water Flood":     0.5,
    "Low-Sal. WF":         0.5,
  },

  // Water treatment flag thresholds
  wt_tds:          50000,  // mg/L — flag if TDS > this AND perm < threshold
  wt_divalents:    1000,   // mg/L — high divalents → scaling risk
  wt_oiw:          100,    // mg/L — Oil-in-Water threshold
  wt_solids:       20,     // mg/L — Suspended solids
  wt_perm:         300,    // mD — below this perm, water quality is critical

  // Strategic Fit weights (must sum to 100)
  potentialWeight:    33,
  costWeight:         33,
  speedWeight:        34,
};

// ─────────────────────────────────────────────────────────────────────────────
// 3. LOGISTICS CHECKLIST QUESTIONS
//    Grouped by method (Yes/Partial/No scoring)
// ─────────────────────────────────────────────────────────────────────────────

export const LOGISTICS_QUESTIONS = {
  "General": [
    { key: "onshore",     label: "Onshore field (vs offshore — EOR more costly/complex offshore)?", softCap: false },
    { key: "infra",       label: "Existing injection infrastructure (wells, flowlines, pumps)?",    softCap: false },
    { key: "wells",       label: "Sufficient well count & pattern spacing achievable?",              softCap: false },
    { key: "access",      label: "Road/logistics access for chemicals, equipment & personnel?",      softCap: false },
    { key: "regulatory",  label: "Regulatory & HSE framework for EOR injection in place?",          softCap: false },
    { key: "wastewater",  label: "Produced water / effluent disposal plan exists?",                 softCap: false },
    { key: "economics",   label: "Project economics evaluated at current oil price?",               softCap: false },
  ],
  "Miscible CO2": [
    { key: "co2_source",  label: "⚠ CO₂ source available within economic distance?", softCap: true  },
    { key: "co2_pipe",    label: "CO₂ pipeline/transport infrastructure feasible?",  softCap: false },
    { key: "co2_compr",   label: "CO₂ capture/compression/dehydration facility feasible?", softCap: false },
    { key: "co2_pres",    label: "Reservoir pressure ≥ MMP feasible (with WI if needed)?", softCap: false },
    { key: "co2_permit",  label: "CO₂ injection regulatory permit obtainable?",     softCap: false },
  ],
  "Immiscible CO2": [
    { key: "ico2_source", label: "⚠ CO₂ source available nearby?",           softCap: true  },
    { key: "ico2_pipe",   label: "CO₂ transport/pipeline feasible?",         softCap: false },
    { key: "ico2_corr",   label: "Corrosion-resistant well completions available?", softCap: false },
    { key: "ico2_recyfl", label: "CO₂ recycling (reinjection) feasible?",   softCap: false },
    { key: "ico2_permit", label: "CO₂ injection regulatory permit obtainable?", softCap: false },
  ],
  "Miscible HC Gas": [
    { key: "hcg_source",  label: "⚠ Lean enriched gas (C2–C6) source available?", softCap: true  },
    { key: "hcg_compr",   label: "Gas compression infrastructure in place?",        softCap: false },
    { key: "hcg_econ",    label: "Gas sales vs. injection economics evaluated?",    softCap: false },
    { key: "hcg_vol",     label: "Sufficient gas volumes for full project life?",   softCap: false },
    { key: "hcg_permit",  label: "HC gas injection permit obtainable?",             softCap: false },
  ],
  "WAG": [
    { key: "wag_gas",     label: "⚠ Gas source available (HC or CO₂)?",       softCap: true  },
    { key: "wag_water",   label: "Fresh/treated water source available?",       softCap: false },
    { key: "wag_equip",   label: "Alternating injection equipment feasible?",   softCap: false },
    { key: "wag_compr",   label: "Gas compression capacity sufficient?",        softCap: false },
    { key: "wag_gas_bt",  label: "Produced gas handling for breakthrough manageable?", softCap: false },
  ],
  "N2 Injection": [
    { key: "n2_plant",    label: "⚠ N₂ generation plant feasible/viable?",      softCap: true  },
    { key: "n2_equip",    label: "High-pressure N₂ injection equipment available?", softCap: false },
    { key: "n2_cap",      label: "Cap rock integrity confirmed (N₂ highly buoyant)?", softCap: false },
    { key: "n2_vent",     label: "Produced N₂ handling/venting plan?",          softCap: false },
    { key: "n2_permit",   label: "N₂ injection regulatory approval obtainable?", softCap: false },
  ],
  "Polymer Flood": [
    { key: "poly_water",  label: "⚠ Water source available (any salinity)?",       softCap: true  },
    { key: "poly_supply", label: "Polymer supply chain established (HPAM/ATBS-variant)?", softCap: false },
    { key: "poly_mix",    label: "Polymer mixing & injection facility feasible?",   softCap: false },
    { key: "poly_pwt",    label: "Produced water treatment & polymer disposal plan?", softCap: false },
    { key: "poly_valid",  label: "Polymer chemistry validated for reservoir T & water composition?", softCap: false },
  ],
  "ASP Flood": [
    { key: "asp_water",   label: "⚠ Water source available?",                       softCap: true  },
    { key: "asp_chem",    label: "Alkali, surfactant & polymer supply secured?",    softCap: false },
    { key: "asp_mix",     label: "Chemical mixing plant and injection facility feasible?", softCap: false },
    { key: "asp_sep",     label: "Produced fluid separation (emulsions) capacity?", softCap: false },
    { key: "asp_scale",   label: "Scale & produced water management plan in place?", softCap: false },
  ],
  "Surfactant-Polymer": [
    { key: "sp_water",    label: "⚠ Fresh/low-TDS water source available?",           softCap: true  },
    { key: "sp_supply",   label: "Surfactant supply & formulation validated by lab?", softCap: false },
    { key: "sp_avail",    label: "Surfactant & molecules commercially available",     softCap: false },
    { key: "sp_emul",     label: "Produced emulsion handling capacity?",             softCap: false },
    { key: "sp_ads",      label: "Surfactant adsorption acceptable for this rock type?", softCap: false },
  ],
  "Steam Flood": [
    { key: "st_water",    label: "⚠ High-quality fresh water (large volumes) available?", softCap: true  },
    { key: "st_gen",      label: "Steam generator (OTSG) capacity & fuel source available?", softCap: false },
    { key: "st_heat",     label: "Acceptable heat losses (depth <1400m, pay >6m)?", softCap: false },
    { key: "st_pwt",      label: "Produced water treatment & recycling facility?",  softCap: false },
    { key: "st_surf",     label: "Surface facilities (separators, insulation) feasible?", softCap: false },
  ],
  "In-Situ Combustion": [
    { key: "isc_air",     label: "⚠ Air compression infrastructure feasible?",        softCap: true  },
    { key: "isc_asph",    label: "Sufficient asphaltene content in oil for coke deposition?", softCap: false },
    { key: "isc_gas",     label: "Produced gas (CO₂,N₂,CO,SO₂) handling system?",  softCap: false },
    { key: "isc_comp",    label: "High-temperature well completions available?",     softCap: false },
    { key: "isc_permit",  label: "Environmental permit for combustion gas management?", softCap: false },
  ],
  "Hot Water Flood": [
    { key: "hw_source",   label: "⚠ Hot water source or dedicated heating system available?", softCap: true  },
    { key: "hw_insul",    label: "Insulated surface lines & wellbore equipment feasible?", softCap: false },
    { key: "hw_pwt",      label: "Produced water treatment capacity available?",    softCap: false },
    { key: "hw_econ",     label: "Energy cost for heating water economically viable?", softCap: false },
    { key: "hw_permit",   label: "Thermal injection permit obtainable?",            softCap: false },
  ],
  "Low-Sal. WF": [
    { key: "ls_water",    label: "⚠ Low-salinity water source (or desalination) feasible?", softCap: true  },
    { key: "ls_wet",      label: "Wettability alteration confirmed by lab corefloods?", softCap: false },
    { key: "ls_compat",   label: "Brine compatibilities verified (injection/connate)?", softCap: false },
    { key: "ls_fines",    label: "Fines migration risk assessed & manageable?",     softCap: false },
    { key: "ls_monit",    label: "Salinity monitoring plan (tracers/sampling) in place?", softCap: false },
  ],
};

// ─────────────────────────────────────────────────────────────────────────────
// 4. SCORING FUNCTIONS
// ─────────────────────────────────────────────────────────────────────────────

/**
 * Score a single parameter value against [min, avg, max] using triangular scoring.
 *
 * Scoring logic (identical to Excel formulas):
 * - Below min:  0 (out of range)
 * - min→avg:    linearly 0→100
 * - avg→max:    linearly 100→0
 * - Above max:  0 (out of range)
 * - max=999:    no upper bound — score stays 100 once value ≥ avg
 *
 * Returns 0–100.
 */
export function scoreParameter(value, { min, avg, max }) {
  if (value === null || value === undefined || isNaN(value)) return null;
  if (max === 999) {
    // No upper limit: flat 100 above avg
    if (value < min) return 0;
    if (value < avg) return ((value - min) / (avg - min)) * 100;
    return 100;
  }
  if (value < min || value > max) return 0;
  if (value <= avg) return ((value - min) / (avg - min)) * 100;
  return ((max - value) / (max - avg)) * 100;
}

/**
 * Compute weighted Technical Score for one method and one field.
 *
 * Returns 0–100. Only parameters with weight > 0 are used.
 * Skips parameters where field has no data (null/undefined).
 */
export function computeTechnicalScore(fieldData, methodKey, criteria) {
  const params = criteria[methodKey];
  let weightedSum = 0;
  let totalWeight = 0;

  for (const [paramKey, spec] of Object.entries(params)) {
    if (spec.weight === 0) continue;
    const value = fieldData[paramKey];
    if (value === null || value === undefined || value === "") continue;
    const raw = scoreParameter(parseFloat(value), spec);
    if (raw === null) continue;
    weightedSum += raw * spec.weight;
    totalWeight += spec.weight * 100;
  }

  if (totalWeight === 0) return 0;
  return (weightedSum / totalWeight) * 100;
}

/**
 * Score the logistics checklist for one method and one field.
 * Yes=100, Partial=50, No=0. Average across all questions.
 * If a soft-cap question is "No", return min(score, logSoftCap).
 */
export function computeLogisticsScore(fieldLogistics, methodKey, generalAnswers, tuning) {
  const methodQs  = LOGISTICS_QUESTIONS[methodKey] || [];
  const generalQs = LOGISTICS_QUESTIONS["General"]  || [];
  const allQs     = [...generalQs, ...methodQs];

  const valueMap = { Yes: 100, Partial: 50, No: 0 };

  let sum = 0;
  let count = 0;
  let softCapTriggered = false;

  for (const q of allQs) {
    const ans = q.softCap
      ? (fieldLogistics[methodKey]?.[q.key] ?? "No")
      : (generalAnswers?.[q.key] ?? fieldLogistics[methodKey]?.[q.key] ?? "No");

    const v = valueMap[ans] ?? 0;
    sum += v;
    count += 1;
    if (q.softCap && ans === "No") softCapTriggered = true;
  }

  const raw = count > 0 ? sum / count : 0;
  return softCapTriggered ? Math.min(raw, tuning.logSoftCap) : raw;
}

/**
 * Determine flags for a field+method combination.
 * Returns array of flag strings: 🚫Frac, 🚫MMP, ⚠Log, 💧WT
 */
export function computeFlags(fieldData, methodKey, techScore, logScore, tuning) {
  const flags = [];
  const GAS_METHODS = ["Miscible CO2", "Miscible HC Gas", "N2 Injection"];

  // MMP flag: gas methods where current reservoir pressure < MMP
  if (GAS_METHODS.includes(methodKey)) {
    const pres = parseFloat(fieldData.pres ?? 0);
    const mmpMap = {
      "Miscible CO2":    parseFloat(fieldData.mmp_co2   ?? 999),
      "Miscible HC Gas": parseFloat(fieldData.mmp_hcgas ?? 999),
      "N2 Injection":    parseFloat(fieldData.mmp_n2    ?? 999),
    };
    const mmp = mmpMap[methodKey];
    if (pres < mmp) flags.push("🚫MMP");
  }

  // Fractured carbonate flag
  if (fieldData.rockType === "Fract. Carb." && methodKey !== "Low-Sal. WF") {
    flags.push("🚫Frac");
  }

  // Logistics flag
  if (logScore <= tuning.logSoftCap + 1) flags.push("⚠Log");

  // Water treatment flag
  const tds      = parseFloat(fieldData.tds_formation ?? 0);
  const divs     = parseFloat(fieldData.divalents     ?? 0);
  const oiw      = parseFloat(fieldData.oiw           ?? 0);
  const solids   = parseFloat(fieldData.suspended     ?? 0);
  const perm     = parseFloat(fieldData.perm          ?? 9999);

  if (
    (tds > tuning.wt_tds && perm < tuning.wt_perm) ||
    divs > tuning.wt_divalents ||
    oiw > tuning.wt_oiw ||
    solids > tuning.wt_solids
  ) {
    flags.push("💧WT");
  }

  return flags;
}

/**
 * Apply rock-type modifier to technical score.
 */
export function applyRockModifier(techScore, fieldData, methodKey, tuning) {
  if (fieldData.rockType === "Fract. Carb." && methodKey !== "Low-Sal. WF") {
    return Math.min(techScore, tuning.fracturedCarbCap);
  }
  if (fieldData.rockType === "Carbonate") {
    const mod = tuning.rockModifiers[methodKey] ?? 1.0;
    return techScore * mod;
  }
  return techScore;
}

/**
 * Apply MMP soft-cap to technical score.
 */
export function applyMMPCap(techScore, fieldData, methodKey, tuning) {
  const GAS_METHODS = ["Miscible CO2", "Miscible HC Gas", "N2 Injection"];
  if (!GAS_METHODS.includes(methodKey)) return techScore;

  const pres = parseFloat(fieldData.pres ?? 0);
  const mmpMap = {
    "Miscible CO2":    parseFloat(fieldData.mmp_co2   ?? 999),
    "Miscible HC Gas": parseFloat(fieldData.mmp_hcgas ?? 999),
    "N2 Injection":    parseFloat(fieldData.mmp_n2    ?? 999),
  };
  const mmp = mmpMap[methodKey];
  if (pres < mmp) return Math.min(techScore, tuning.techSoftCap);
  return techScore;
}

// ─────────────────────────────────────────────────────────────────────────────
// 5. STRATEGIC FIT SCORING
//    Independent from technical/logistics. Ranks methods by value delivery.
//    Potential × Cost × Speed, weighted average.
// ─────────────────────────────────────────────────────────────────────────────

/**
 * Strategic Fit subscores: Potential, Cost, Speed.
 *
 * These are field-level inputs (OOIP, RF, wells, etc.) combined with
 * method-level characteristics (capital cost tier, time-to-production tier).
 *
 * Method profiles: base scores representing "how good is this method" on each axis.
 * Field adjustments applied on top.
 */
const METHOD_PROFILES = {
  //                              Potential  Cost  Speed   (base 0–100)
  "Miscible CO2":        { pot: 80, cost: 55, speed: 60 },
  "Immiscible CO2":      { pot: 75, cost: 58, speed: 62 },
  "Miscible HC Gas":     { pot: 78, cost: 60, speed: 60 },
  "WAG":                 { pot: 82, cost: 58, speed: 62 },
  "N2 Injection":        { pot: 76, cost: 56, speed: 60 },
  "Polymer Flood":       { pot: 70, cost: 62, speed: 65 },
  "ASP Flood":           { pot: 78, cost: 55, speed: 66 },
  "Surfactant-Polymer":  { pot: 78, cost: 55, speed: 65 },
  "Steam Flood":         { pot: 72, cost: 45, speed: 58 },
  "In-Situ Combustion":  { pot: 74, cost: 42, speed: 55 },
  "Hot Water Flood":     { pot: 70, cost: 48, speed: 58 },
  "Low-Sal. WF":         { pot: 75, cost: 65, speed: 68 },
};

/**
 * Compute Strategic Fit score for one method and one field.
 *
 * Field adjustments:
 * - Potential adjusted by OOIP size, current RF (more room = higher potential),
 *   oil viscosity match to method
 * - Cost adjusted by well count (more existing infrastructure = cheaper),
 *   onshore/offshore
 * - Speed adjusted by existing injectors, well spacing
 */
export function computeStrategicFit(fieldData, methodKey, tuning) {
  const profile = METHOD_PROFILES[methodKey];
  if (!profile) return { potential: 0, cost: 0, speed: 0, fit: 0 };

  const ooip      = parseFloat(fieldData.ooip     ?? 0);
  const rf        = parseFloat(fieldData.rf        ?? 50) / 100;
  const wells     = parseFloat(fieldData.wells     ?? 0);
  const injectors = parseFloat(fieldData.injectors ?? 0);
  const spacing   = parseFloat(fieldData.spacing   ?? 300);

  // Potential: base + remaining recovery upside
  const recoveryRoom = Math.max(0, 1 - rf); // 0→1
  const ooipBonus    = Math.min(20, ooip / 200);  // bigger field → more upside
  let potential = profile.pot * (0.5 + 0.5 * recoveryRoom) + ooipBonus;
  potential = Math.min(100, Math.max(0, potential));

  // Cost: base + infrastructure penalty/bonus
  const infraBonus = Math.min(15, (wells / 50) * 10);
  let cost = profile.cost + infraBonus;
  cost = Math.min(100, Math.max(0, cost));

  // Speed: base + existing injectors bonus
  const injBonus = injectors > 0 ? Math.min(15, (injectors / wells) * 20) : 0;
  const spacingPenalty = spacing > 400 ? -5 : 0;
  let speed = profile.speed + injBonus + spacingPenalty;
  speed = Math.min(100, Math.max(0, speed));

  // Weighted average
  const fit = (
    potential * tuning.potentialWeight +
    cost      * tuning.costWeight +
    speed     * tuning.speedWeight
  ) / (tuning.potentialWeight + tuning.costWeight + tuning.speedWeight);

  return { potential, cost, speed, fit };
}

// ─────────────────────────────────────────────────────────────────────────────
// 6. FULL SCREENING — compute all scores for one field
// ─────────────────────────────────────────────────────────────────────────────

/**
 * Run full EOR screening for a single field.
 *
 * @param {Object} fieldData      - Reservoir input (keyed by parameter)
 * @param {Object} fieldLogistics - Logistics answers {methodKey: {questionKey: "Yes"|"Partial"|"No"}}
 * @param {Object} generalLogistics - General logistics answers {questionKey: "Yes"|"Partial"|"No"}
 * @param {Object} criteria       - Screening criteria (default: SCREENING_CRITERIA)
 * @param {Object} tuning         - Tuning panel values (default: DEFAULT_TUNING)
 *
 * @returns {Object} results by method
 */
export function screenField(
  fieldData,
  fieldLogistics    = {},
  generalLogistics  = {},
  criteria          = SCREENING_CRITERIA,
  tuning            = DEFAULT_TUNING
) {
  const results = {};

  for (const method of EOR_METHODS) {
    // Technical score
    let techRaw = computeTechnicalScore(fieldData, method, criteria);
    techRaw = applyRockModifier(techRaw, fieldData, method, tuning);
    techRaw = applyMMPCap(techRaw, fieldData, method, tuning);
    const tech = Math.max(0, Math.min(100, techRaw));

    // Logistics score
    const log = computeLogisticsScore(fieldLogistics, method, generalLogistics, tuning);

    // Combined score
    const combined = (tech * tuning.techWeight + log * tuning.logWeight) / 100;

    // Flags
    const flags = computeFlags(fieldData, method, tech, log, tuning);

    // Strategic Fit
    const strategic = computeStrategicFit(fieldData, method, tuning);

    results[method] = {
      tech:      Math.round(tech * 10) / 10,
      log:       Math.round(log  * 10) / 10,
      combined:  Math.round(combined * 10) / 10,
      strategic: {
        potential: Math.round(strategic.potential * 10) / 10,
        cost:      Math.round(strategic.cost      * 10) / 10,
        speed:     Math.round(strategic.speed     * 10) / 10,
        fit:       Math.round(strategic.fit       * 10) / 10,
      },
      flags,
      isCandidate: combined >= tuning.passThreshold,
    };
  }

  return results;
}

/**
 * Get top N ranked methods for a field, by combined score.
 */
export function topMethods(results, n = 3) {
  return Object.entries(results)
    .sort((a, b) => b[1].combined - a[1].combined)
    .slice(0, n)
    .map(([method, scores]) => ({ method, ...scores }));
}
