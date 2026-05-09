# RadioB VIN Scan — Documentation

**Version:** v0.8.0  
**File:** `index.html` (single-file web app)

---

## Overview

RadioB VIN Scan is a mobile-friendly, single-page web application for decoding Vehicle Identification Numbers (VINs). It lets users scan, enter, or look up a VIN to retrieve vehicle specifications from the NHTSA database, search for used OEM parts on eBay, and find donor vehicles at local salvage yards via Row52.

---

## Features

### 1. VIN Input Methods

#### Google Lens (Camera Scan)
- Opens Google Lens to scan a VIN barcode or plate with the device camera.
- On Android, attempts to launch the native Lens app via an intent URL; falls back to the Lens website after 1.5 seconds.
- On other platforms, opens `https://lens.google.com` directly.
- Workflow: scan the VIN with Lens → copy the text → paste into the manual input field.

#### Manual Entry
- Text input accepts up to 17 characters, auto-uppercased.
- On paste, automatically strips non-alphanumeric characters and triggers decoding if the result is exactly 17 characters.
- Press Enter or click **Decode** to submit.

#### Sample VINs
- Four pre-loaded sample chips for testing (2021 Honda, 2013 Ford F-150, 2018 Toyota, 2015 Jeep).
- Clicking a chip populates the input and triggers decoding immediately.

---

### 2. VIN Decoding

**API:** [NHTSA VPIC — DecodeVinValuesExtended](https://vpic.nhtsa.dot.gov/api/)

**Validation (client-side, before API call):**
- Must be exactly 17 characters.
- Cannot contain the letters I, O, or Q (illegal in VINs).

**Displayed vehicle data:**
| Field | NHTSA Key |
|---|---|
| Year | `ModelYear` |
| Make | `Make` |
| Model | `Model` |
| Trim / Series | `Trim` or `Series` |
| Body Style | `BodyClass` |
| Drive Type | `DriveType` |
| Fuel Type | `FuelTypePrimary` |
| Engine | Composed from `DisplacementL`, `EngineCylinders`, `EngineConfiguration`, `Turbo`, `EngineHP` |
| Transmission | `TransmissionStyle` |
| Doors / Seating | `Doors`, `Seats` |
| Vehicle Type | `VehicleType` |
| Manufacturer | `ManufacturerName` |
| Assembly Plant | `PlantCity`, `PlantState`, `PlantCountry` |
| GVWR | `GVWR` |
| Electrification | `ElectrificationLevel` |

Fields with no value, `"Not Applicable"`, `"0"`, or `""` are hidden automatically.

---

### 3. eBay Parts Search

After a VIN is decoded, a parts panel appears with pre-built eBay search buttons for common OEM parts categories:

| Category | eBay Search Terms |
|---|---|
| Engine | `engine OEM` |
| Transmission | `transmission OEM` |
| Modules | `OEM module` |
| Infotainment / Nav | `infotainment navigation radio OEM` |
| Headlights & Taillights | `headlight taillight assembly LED` |
| Seats & Interior Trim | `seat interior trim` |
| Catalytic Converter | `catalytic converter OEM` |
| Doors, Fenders & Hoods | `door fender hood` |

Each button searches **eBay Motors (category 6030)** for **sold/completed listings**, **used condition**, sorted **high to low price**.

A **custom part search** field is also provided — enter any part name, press Enter or the search icon, and it opens the same eBay filter with that term.

---

### 4. Search History

- Stores up to **20** decoded VINs in `localStorage` under the key `radiob_vin_history`.
- Displays vehicle name, VIN, and relative time (e.g., "3h ago").
- Clicking a history item re-decodes that VIN and scrolls to the top.
- Individual items can be removed; **Clear All** wipes the entire history.
- History panel is hidden when empty.

---

### 5. Row52 Yard Inventory

Searches [row52.com](https://row52.com) for matching donor vehicles at nearby salvage yards.

**Fields:**
| Field | Details |
|---|---|
| Make | Dropdown populated from a built-in make/ID map (60+ makes) |
| Year | Optional; accepts a single year (`2010`) or range (`2008-2012`) |
| ZIP Code | Required 5-digit ZIP; saved to `localStorage` (key: `radiob_row52_zip`) |
| Distance | 10 / 25 / **50** (default) / 100 / 200 miles |

**Actions:**
- **Search Row52** — loads results in an inline iframe below the form.
- **↗ (open in new tab)** — opens the same URL in a new browser tab.
- If the iframe fails to load (Row52 blocks embedding), a fallback "Open in new tab" link appears after 4 seconds.

**Auto-prefill:** When a VIN is decoded, the Row52 panel automatically populates the Make and Year fields from the decoded vehicle data.

**Make matching:** NHTSA make names are normalized and matched against the Row52 make list. Aliases handled: `vw → volkswagen`, `mb → mercedesbenz`, `chevy → chevrolet`. Falls back to prefix matching for names like "Mercedes-Benz USA".

---

## UI / Layout

- Max width: **480px** (560px on screens ≥ 600px wide).
- Responsive landscape layout: scanner section becomes side-by-side at `orientation: landscape` and `max-height: 500px`.
- CSS custom properties for theming:

| Variable | Default | Usage |
|---|---|---|
| `--bg` | `#f5f5f5` | Page background |
| `--surface` | `#ffffff` | Cards |
| `--surface2` | `#f0f0f0` | Inputs, secondary buttons |
| `--border` | `#ddd` | Borders |
| `--accent` | `#1a6fe8` | Primary actions, links |
| `--text` | `#1a1a1a` | Body text |
| `--muted` | `#888` | Labels, hints |
| `--success` | `#2e7d32` | Decoded badge |
| `--error` | `#c62828` | Error states |

---

## Dependencies

| Library | Version | CDN | Purpose |
|---|---|---|---|
| ZXing | 0.19.1 | unpkg | Barcode decoding (Code 39 / Code 128) |
| Tesseract.js | 2.1.5 | unpkg | OCR text recognition |

Both libraries are loaded from CDN; no build step or local installation required.

> **Note:** The barcode scanner and OCR photo capture UI elements (`.photo-viewport`, mode toggle buttons, scan/capture buttons) are referenced in CSS and JavaScript but their HTML is not present in the current `index.html`. These features are scaffolded for a future implementation.

---

## Data Storage (localStorage)

| Key | Type | Contents |
|---|---|---|
| `radiob_vin_history` | JSON array | `{ vin, label, ts }` objects, max 20 items |
| `radiob_row52_zip` | String | Last-used ZIP code for Row52 search |

---

## Key Functions

| Function | Description |
|---|---|
| `decodeVIN(vin?)` | Validates and calls the NHTSA API; shows result or error |
| `showVehicleResult(vin, r)` | Renders the decoded vehicle card with specs and eBay buttons |
| `openGoogleLens()` | Opens Google Lens (Android intent or web fallback) |
| `openEbayPart(year, make, model, term)` | Builds and opens an eBay sold-listing search URL |
| `searchCustomPart(year, make, model)` | Reads the custom part input and calls `openEbayPart` |
| `addToHistory(vin, label)` | Prepends a VIN to localStorage history |
| `renderHistory()` | Re-renders the history panel from localStorage |
| `searchRow52(openInNewTab?)` | Builds Row52 URL and loads it in iframe or new tab |
| `row52PrefillFromVehicle(year, make)` | Pre-populates Row52 form after a VIN decode |
| `initRow52Panel()` | Populates the make dropdown and restores saved ZIP on load |
| `buildEngine(r)` | Composes a human-readable engine string from NHTSA fields |
| `buildPlant(r)` | Composes the assembly plant location string |
| `gv(r, key)` | Helper: returns a field value or `null` if empty/not applicable |
| `timeAgo(ts)` | Returns a human-readable relative time string |

---

## Limitations & Known Issues

- **Barcode / OCR photo capture:** The JavaScript handlers (`handleBarcodePhoto`, `handleOcrPhoto`, `setMode`) and CSS for this feature are present, but the corresponding HTML elements (file input, mode toggle buttons, viewport) are absent. These inputs do nothing without the UI wiring.
- **iframe embedding:** Row52 may send `X-Frame-Options: SAMEORIGIN`, causing the inline iframe to remain blank. The fallback link appears after 4 seconds.
- **NHTSA API rate limits:** No caching or rate-limit handling is implemented; heavy use may be throttled by the API.
- **No offline support:** No service worker or local caching; the app requires an internet connection.
