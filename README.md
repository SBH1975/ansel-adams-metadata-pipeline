# Ansel Adams Digital Collection: Metadata Cataloging, Crosswalking & METS Packaging Workflow

This repository documents the end-to-end archival data engineering pipeline used to catalog, normalize, crosswalk, and package digital assets from Wikimedia Commons featuring the photography of Ansel Adams. The workflow spans basic bibliographic capture in **Simple Dublin Core (DCMES)**, full bibliographic description using **RDA (Resource Description and Access)** and **Library of Congress Subject Headings (LCSH)** in **MARC 21**, schema crosswalking into **Qualified Dublin Core (QDC)**, and final packaging of archival masters, web derivatives, and descriptive metadata inside valid **METS (Metadata Encoding and Transmission Standard) XML** wrappers.

---

## Table of Contents
1. [Overview & Selected Assets](#overview--selected-assets)
2. [Step 1: Simple Dublin Core Baseline](#step-1-simple-dublin-core-baseline)
3. [Step 2: Full MARC 21 / RDA / LCSH Cataloging](#step-2-full-marc-21--rda--lcsh-cataloging)
4. [Step 3: MARC to Qualified Dublin Core Crosswalk](#step-3-marc-to-qualified-dublin-core-crosswalk)
5. [Step 4: METS XML Packaging & Schema Architecture](#step-4-mets-xml-packaging--schema-architecture)
6. [Repository Structure & File Manifest](#repository-structure--file-manifest)
7. [Validation & Reproducibility](#validation--reproducibility)

---

## Overview & Selected Assets

Five photographs by Ansel Adams were selected from Wikimedia Commons and public institutional repositories (National Archives and Records Administration, Library of Congress, Saint Louis Art Museum):

1. **The Tetons and the Snake River** (1942) – Grand Teton National Park, Wyoming (NARA NAID: `519904`)
2. **Monolith, the Face of Half Dome** (1927) – Yosemite National Park, California (*Parmelian Prints of the High Sierras*)
3. **Farm workers and Mt. Williamson** (1943) – Manzanar War Relocation Center, California (LOC `LOT 10479-4`)
4. **Clearing Winter Storm, Yosemite Valley** (1937) – Yosemite National Park, California (Saint Louis Art Museum ID: `36452`)
5. **View of valley from mountain, Canyon de Chelly National Monument** (1942) – Apache County, Arizona (NARA NAID: `519852`)

---

## Step 1: Simple Dublin Core Baseline

The initial cataloging phase establishes an interchange baseline by populating the **15 Simple Dublin Core Metadata Elements (ISO 15836 / ANSI/NISO Z39.85)**:

* `Title`
* `Creator`
* `Subject`
* `Description`
* `Publisher`
* `Contributor`
* `Date`
* `Type`
* `Format`
* `Identifier`
* `Source`
* `Language`
* `Relation`
* `Coverage`
* `Rights`

### Data Capture Rules:
* **Creator / Contributor**: Inverted personal author entries (`Adams, Ansel`) or government departments (`U.S. National Park Service`).
* **Date**: ISO 8601 (`YYYY` or `YYYY-MM-DD`).
* **Format**: Standard Internet Media Types (`image/jpeg`).
* **Rights**: Clear copyright notices indicating federal public domain status, expiration prior to 1931, or non-renewal under U.S. copyright law.

---

## Step 2: Full MARC 21 / RDA / LCSH Cataloging

To support deep archival discovery and ILS/LSP ingestion, each asset was cataloged as a full MARC 21 Bibliographic record under **RDA (Resource Description and Access)** guidelines with authorized Library of Congress controlled vocabularies:

### Structural Encoding:
* **Leader & Fixed Fields (`007`, `008`)**:
  * `007` encoded for non-projected graphics (`k`), category specific (`j`), color/monochrome (`b` or `c`), primary support (`o`).
  * `008` coded for visual materials (`kneng d`), date type `s`, and US state jurisdictions (`wyu`, `cau`, `azu`).
* **Name Authority (`100`, `710`)**: Formatted per LC Name Authority File (`Adams, Ansel, 1902-1984, photographer.`) with RDA relator terms (`$e`).
* **Title & Statement of Responsibility (`245`)**: Title transcribed with non-filing indicator calculations (e.g., `245 14 $a The Tetons...` where `14` skips "The ").
* **Publication & Production (`264`)**: Indicator 2 specifies status (`\0` for creation date, `\1` for commercial publication).
* **RDA Carrier & Content Types (`336`, `337`, `338`)**:
  * Content: `336 $a two-dimensional image $b sti $2 rdacont`
  * Media: `337 $a unmediated $b n $2 rdamedia`
  * Carrier: `338 $a sheet $b nb $2 rdatarier`
* **Subject Analysis (`650`, `651`)**:
  * Topical terms drawn strictly from **LCSH** (e.g., `Landscape photography--Wyoming--Grand Teton National Park`).
  * Geographic headings formatted with authorized qualifiers (`Yosemite Valley (Calif.)`, `Snake River (Wyo.-Wash.)`).
* **Parent Linkages & Electronic Access (`773`, `856`)**: Host portfolio/collection relationships (`773`) and resolution URIs to repository master files (`856 40 $u`).

---

## Step 3: MARC to Qualified Dublin Core Crosswalk

The granular MARC 21 fields were mapped to **Qualified Dublin Core (QDC / DCTERMS)** elements and refinements, reconciling library-grade cataloging with modern web-harvestable linked data schemas:

| Qualified Dublin Core Term | Qualified Namespace | Source MARC 21 Field / Subfield |
| :--- | :--- | :--- |
| **`title`** | `dcterms:title` | `245 $a` (minus non-filing characters) |
| **`title.alternative`** | `dcterms:alternative` | `246 $a` / `500` variant title note |
| **`creator`** | `dcterms:creator` | `100 $a, $d` (LC Name Authority format) |
| **`subject.lcsh`** | `dcterms:subject` | `650 $a, $z`, `651 $a, $v` concatenated |
| **`description.abstract`** | `dcterms:abstract` | `520 $a`, `500 $a`, `536 $a` project notes |
| **`publisher`** | `dcterms:publisher` | `264 \1 $b` or issuing agency `710 $a` |
| **`contributor`** | `dcterms:contributor` | `700`, `710` non-publishing contributors |
| **`date.created`** | `dcterms:created` | `264 \0 $c` |
| **`date.issued`** | `dcterms:issued` | `264 \1 $c` |
| **`type`** | `dcterms:type` | `336 $a` mapped to DCMI Type (`StillImage`) |
| **`format.medium`** | `dcterms:medium` | `300 $b` physical medium + `image/jpeg` |
| **`format.extent`** | `dcterms:extent` | `300 $a`, `300 $c` dimensions |
| **`identifier.uri`** | `dcterms:identifier` | `856 $u`, NAID, or LC control numbers |
| **`relation.isPartOf`** | `dcterms:isPartOf` | `773 $t, $w` parent collections |
| **`language.iso639-2`** | `dcterms:language` | `008/35-37` (`eng`) |
| **`coverage.spatial`** | `dcterms:spatial` | `651 $a`, `651 $z` geographic subjects |
| **`rights.accessRights`** | `dcterms:accessRights` | `540 $a` rights statement |

---

## Step 4: METS XML Packaging & Schema Architecture

Each photograph is wrapped in an individual, production-grade **METS (Metadata Encoding & Transmission Standard) v1.12** container designed for long-term OAIS archival storage (AIP / DIP).
