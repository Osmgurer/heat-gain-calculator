# Heat Gain Calculator

## Overview
A single-file web application for calculating total heat load (W and BTU/h) for rooms/spaces such as wine cellars. Version 3 with tabbed layout.

## Architecture
- **index.html** - Complete application (HTML + CSS + JS embedded)
- **server.js** - Node.js HTTP server on port 5000

## Layout (V4 Tabbed)
- **App Header**: Title only (minimal)
- **Tab Bar**: 8 tabs — Project, Geometry, Temperatures, Solid U-Values, Glass Config, Internal Gains, Solar/SHGC, Results
- **Project tab**: Contains Project Information (name/date), Project Files (save/load/export), Global Actions (reset/copy summary)

## Features
- **Conduction loads**: Floor, ceiling, 4 walls, glass areas per wall
- **Internal heat gains**: People, lighting (continuous/intermittent), equipment, door opening loss, wine bottle turnover
- **Dual unit system**: Metric/Imperial toggle with simultaneous display of both units
- **Per-wall temperatures**: Individual outside temperatures for each wall (Wall 1-4) and each glass wall (Glass Wall 1-4), with auto-sync to exterior temp and manual override support
- **Bulk apply buttons**: "Apply Exterior to all walls" for walls group and glass group separately
- **Geometry**: Room dimensions with floor area mode toggle (Rectangular/Irregular), wall length mode toggle (Individual/Total), auto-fill wall lengths from room dimensions (Width→Wall 2/4, Length→Wall 1/3), glass per wall with height override
- **U-values & Insulation**: Two-part base assembly + insulation system for solid surfaces, glazing configuration for glass
- **Validation**: Inline errors, glass vs wall length checks, input range enforcement
- **Disregard toggles**: "Disregard Solar Heat Gain" (default ON) in Solar tab; "Disregard the Base Assembly" (default ON) per surface in Solid U-Values tab
- **Results**: Detailed breakdown table with area/ΔT info, conduction subtotal, internal gains subtotal, solar subtotal (with disregard indicator), and grand total
- **Summary PDF**: One-page A4 PDF report via jsPDF (CDN), includes all sections, Grand Total prominent, browser save dialog
- **UX**: Reset defaults, tabbed layout, project save/load/export
- **Project Management**: Save to JSON, load from JSON, export as offline HTML package with embedded state

## Save/Load System
- `collectState()`: Gathers all form values, dropdowns, checkboxes, equipment rows, surfManual/uAutoMode flags
- `applyState(s)`: Restores full state from a JSON object, including re-triggering UI updates
- **File System Access API** (primary path, Chrome/Edge): `saveProject()` and `exportOffline()` use `showSaveFilePicker()`, `loadProject()` uses `showOpenFilePicker()` — lets user choose folder
- **Fallback**: If API unavailable, shows a modal explaining limitations with "Download Anyway" or "Cancel" options; load falls back to hidden `<input type="file">`
- **iframe detection**: `isInIframe()` shows warning banner + "Open in New Tab" button when embedded
- **File Access Status**: `#fileAccessStatus` shows "File picker supported: Yes/No" with tip
- `exportOffline()`: Saves the entire HTML page with embedded `__EMBEDDED_PROJECT__` state variable; on load, auto-applies state
- `updateFileAccessStatus()`: Called in `init()` to display support status and iframe warning

## U-Value System (V2)
### Solid Surfaces (Floor, Ceiling, Walls)
Two-part assembly system:
1. **Base Assembly** dropdown per surface:
   - Concrete slab (solid, k=1.40, default 200mm)
   - Plywood sandwich panel (sandwich: ply/core/ply, k_ply=0.13, k_core=0.16)
   - Drywall with air cavity (sandwich_floor for floor; sandwich for ceiling/walls)
   - Brick (solid, k=0.72, default 220mm)
   - Concrete block (solid, k=1.05, default 200mm)
   - Stone (solid, k=2.30, default 300mm)
   - Custom (manual k entry)

2. **Insulation** dropdown per surface:
   - XPS (k=0.034), EPS (k=0.038), PIR (k=0.022), PU spray (k=0.024)
   - Stone wool (k=0.035), Aerogel (k=0.015)
   - None (no insulation layer)
   - Custom (manual k entry via `insCustomK_{Surface}` field)

- Auto-calculated U: U = 1 / (Rsi + R_base + R_insulation + Rse)
  - Rsi = 0.13, Rse = 0.04 m²·K/W
- Manual U edit → "Custom U" mode; re-selecting base/insulation resumes auto mode

### Glazing (Glass) — EN 673 / ISO 10292 Style Model
- Configuration: Single, Double (IGU), Triple (IGU), Custom
- Low-E Coating: modifies gap emissivity (ε_clear=0.84 → ε_lowE=0.04), NOT a U multiplier
- Laminated Glass: PVB added as real R layer (ply+PVB+ply), NOT a penalty multiplier
- PVB interlayer: 0.38/0.76/1.14/1.54 mm, k_pvb=0.20 W/m·K
- Gap model: R_gap = 1/(h_c + h_r), h_r from linearized radiation with emissivity, h_c from Ra/Nu correlation
- Argon properties: k=0.016, ν=2.2e-5, α=2.0e-5 m²/s
- Film coefficients: hi=8.0 (Rsi=0.125), ho=25.0 (Rse=0.04)
- **"Use Environmental Conditions" toggle** (#glazeUseEnvTemps): OFF=standard Ti=20/To=0°C, ON=uses app's Int/Ext temps
- **Glass U Debug Mode**: Toggle (#glassDebugToggle) exposes all intermediate values (Ti/To/TmK, hi/ho/Rsi/Rse, per-layer R including PVB, per-gap ε₁/ε₂/h_r/Ra/Nu/h_c/R_gap, summary). Reference U input (#glassRefU) shows Δ% comparison. Read-only — does not affect calculations.

## Solar Heat Gain (SHGC) Module
- 8 region presets + Custom
- Design month selector; irradiance lookup per region/month/orientation
- SHGC auto-derived from glass config; per-wall Auto/Manual mode
- Shading factors: Exterior (Fext) and Interior (Fint), 0-1, per wall
- Q_solar = A_glass × Irradiance × SHGC × Fext × Fint
- Grand Total = Conduction + Internal + Solar

## Tab Names → IDs
- Project: `tab-project`
- Geometry: `tab-geometry`
- Temperatures: `tab-temperatures`
- Solid U-Values: `tab-solidU`
- Glass Config: `tab-glass`
- Internal Gains: `tab-internal`
- Solar / SHGC: `tab-solar`
- Results: `tab-results`

## Key Formulas
- Conduction: Q = U × A × ΔT (per surface)
- People: avg W = count × W/person × (hours/24)
- Lighting continuous: avg W = power × (hours/24)
- Lighting intermittent: avg W = power × (duty%/100)
- LED Bar mode: P = length(m) × W/m × (dim%/100)
- Equipment: sum of (power × hours/24) per item
- Door opening: W = openings/hr × Wh/opening
- Wine turnover: avg W = (bottles × volume × density × cp × ΔT) / 86400

## Unit Conversions
- 1 W = 3.412141633 BTU/h
- 1 W/m²·K = 0.1761101838 BTU/h·ft²·°F
- 1 m = 3.28084 ft
- 1 m² = 10.7639 ft²
- Thickness: 1 in = 25.4 mm
