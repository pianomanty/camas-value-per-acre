# Migration Plan: Chicago to Camas, WA

This document outlines the steps required to adapt the "Value Per Acre" project from Chicago/Cook County to the city of Camas, Washington.

## 1. Data Acquisition (The Most Critical Step)

The entire pipeline depends on finding and downloading datasets for Camas, WA that match the required schema.

### 1.1 Parcel Boundaries (Shapefile/GeoJSON)
- **Goal**: Find the official parcel/cadastral boundaries for Camas, WA.
- **Source**: Clark County GIS (or Washington State GIS Clearinghouse).
- **Action**: Download the latest parcel layer in a format compatible with `GeoPandas` (Shapefile or GeoJSON).

### 1.2 Property Assessment/Tax Data (CSV)
- **Goal**: Find property values and tax information.
- **Source**: Clark County Assessor's Office.
- **Action**: Identify a dataset containing:
    - Property Identifier (equivalent to Cook County's PIN).
    - Assessed Value or Market Value.
    - Property Class/Type (Residential, Commercial, etc.).
    - Property Address (to link with parcel geometry).

### 1.3 City Boundary (GeoJSON)
- **Goal**: A polygon representing the city limits of Camas.
- **Source**: Clark County GIS or US Census Bureau (TIGER/Line).
- **Action**: Download the Camas city boundary GeoJSON.

## 2. Pipeline Adaptation (Python/Bash)

Once data is acquired, the scripts must be updated to handle the new data formats and logic.

### 2.1 `scripts/01_process_parcel_data.py`
- **Update**: Change the input path to point to the Clark County parcel shapefile.
- **Update**: Change the boundary path to point to the Camas boundary GeoJSON.
- **Update**: Update the region argument to `camas`.

### 2.2 `scripts/02_process_assessor_data.py`
- **Update**: Update the input path to the new Clark County assessment CSV.
- **Update**: **Crucial**: Inspect the new CSV columns. You will likely need to rename columns to match the expected `tax_year`, `pin`, `board_tot`, and `class` names used in the current script.
- **Update**: Re-evaluate the logic for cleaning monetary fields (e.g., if they don't use `$` or `,`).

### 2.3 `scripts/03_process_address_data.py`
- **Update**: Update the input path to the new Clark County address CSV.
- **Update**: **Crucial**: Inspect the new CSV columns. You will need to map the new address columns (Street, City, State, Zip) to the `full_address` construction logic.

### 2.4 `scripts/04_join_parcel_data.py`
- **Update**: **Crucial**: The "Market Value Multiplier" logic (`get_market_value_multiplier`) is highly specific to Cook County's assessment classes. You **must** research Clark County's assessment ratios and rewrite this function.
- **Update**: Update the PIN standardization logic (`clean_pin_10digit`) to match the format used by Clark County (e.g., if they use a different length or structure).
- **Update**: Update the property class mapping to match Clark County's classification system.

### 2.5 `scripts/05_generate_tiles.sh`
- **Update**: Update the region argument to `camas`.

## 3. Web Application Adaptation (HTML/JS)

The frontend needs to be updated to reflect the new geographic context and data links.

### 3.1 `web/js/app.js`
- **Update**: Update the `TILES` object to point to `camas_parcels.pmtiles` and `caminty_parcels_hq.pmtiles`.
- **Update**: Update the `buildValuePopupHtml` function:
    - Update the "Source" link to point to the Clark County Assessor's website instead of Cook County.
    - Update any text references to "Cook County" or "Chicago".
- **Update**: Update the `currentExtent` logic if you decide to include "Clark County" as an option alongside "Camas".

### 3.2 `web/index.html` & `web/css/style.css`
- **Update**: Update all text, headings, and descriptions to refer to Camas, WA.
- **Update**: Update the legend and any transit/overlay references (remove CTA, add relevant Camas/Clark County transit if applicable).

## 4. Summary of Required Research

| Item | Research Task |
| :--- | :--- |
| **Identifier** | What is the unique property ID format for Clark County? |
| **Assessment** | What are the assessment ratios for different property classes in Clark County? |
| **Data Source** | Where is the most reliable, downloadable source for Clark County parcel and tax data? |
| **Address Format** | How are addresses structured in the Clark County dataset? |
