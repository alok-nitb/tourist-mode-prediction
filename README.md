# Tourist Mode Predictor

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23067833.svg)](https://doi.org/10.5281/zenodo.23067833)

A single-page React application that predicts a tourist's transport mode behavior — and related satisfaction, safety, and improvement insights — from three simple demographic inputs: **gender, age group, and monthly income**. Predictions are derived entirely from a primary survey of **107 tourist respondents** and run fully client-side, with a one-click export to a formatted PDF report.

This project was built as a way to turn a raw tourist-transport survey into something interactive and explorable, rather than a static spreadsheet of averages.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Available Scripts](#available-scripts)
- [How to Use the App](#how-to-use-the-app)
- [How It Works](#how-it-works)
- [Project Structure](#project-structure)
- [Dataset & Variables](#dataset--variables)
- [Data Anonymisation & Ethics](#data-anonymisation--ethics)
- [Data & Code Availability Statement](#data--code-availability-statement)
- [Limitations](#limitations)
- [Possible Improvements](#possible-improvements)
- [Notes for Future Me / Contributors](#notes-for-future-me--contributors)
- [License](#license)

---

## Overview

Tourists moving around an unfamiliar city make transport choices — private car, rental cab, auto-rickshaw, two-wheeler, city bus, walking — based on habits, budget, and perceived safety that often correlate with demographics. This app lets anyone pick a demographic profile and instantly see:

- what mode that segment of tourists is *most likely* to use,
- how satisfied they tend to be with public transport,
- whether gender shapes their mode choice and safety perception,
- whether specific transport improvements would change their behavior, and
- what real, free-text suggestions tourists in that segment gave.

Everything shown is backed by actual survey respondents, not a generic guess — and the app always tells you how many respondents and which demographic tier the shown result is based on.

## Problem Statement

Given: a tourist survey with demographic fields (gender, age group, income bracket) and a set of behavioral/attitudinal responses (mode choice, satisfaction ratings, scenario answers, suggestions).

Goal: build a tool that, given a new combination of gender + age group + income bracket, returns the most representative behavioral profile from the survey — even when that *exact* combination has few or zero respondents — and present it in a clear, explorable UI with a shareable report output.

## Features

- **Three-input predictor** — pick Gender, Age Group, and Monthly Income from simple segmented controls (no free text, no ambiguous inputs).
- **Cascading demographic match** — falls back gracefully from an exact 3-way match down to broader 2-way and 1-way matches when the exact combination is under-sampled, and always shows which tier matched and on how many respondents it's based.
- **Mode choice behavior panel** — primary mode, most frequent mode category, arrival mode, typical travel time & cost, and the stated main reason for the choice.
- **Public transport satisfaction panel** — 8 satisfaction dimensions (availability, affordability, comfort, safety, accessibility, cleanliness, staff behavior, language/signage) plus an overall score, rendered as color-coded star/bar ratings.
- **Gender-based travel behavior panel** — whether gender influences mode choice for that segment, and how safe public transport is perceived to be.
- **Scenario response panel** — 8 "would you switch to public transport if…" yes/no/not-sure answers (single-route connectivity, daily pass pricing, higher frequency, fare-for-comfort tradeoff, live-tracking app, walkable stop spacing, clear tourist info, eco-friendly promotion).
- **Suggestion feed** — real free-text improvement suggestions from survey respondents, filtered to the selected segment.
- **One-click PDF export** — generates a polished, dark-themed, multi-page PDF of the full prediction using `jsPDF`, entirely in the browser.
- **No backend required** — the whole thing runs as static, client-side React; it can be hosted on any static file host (Vercel, Netlify, GitHub Pages, etc.).

## Tech Stack

| Layer | Technology |
|---|---|
| UI framework | React 19 |
| Build tool / dev server | Vite 8 |
| PDF generation | jsPDF |
| Linting | ESLint 9 (flat config) + `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh` |
| Styling | Plain CSS (no UI framework) |
| Data source | Excel survey export (`.xlsx`), pre-aggregated into JSON |

No state-management library, router, or CSS framework is used — the app is intentionally small and dependency-light.

## Prerequisites

- **Node.js** version 18 or later
- **npm** (comes with Node.js) — or swap the commands below for `yarn`/`pnpm` equivalents if preferred

## Installation & Setup

```bash
# 1. clone the repository
git clone <repo-url>
cd tourist-mode-prediction

# 2. install dependencies
npm install

# 3. start the development server
npm run dev
```

The dev server will print a local URL (typically `http://localhost:5173`) — open it in a browser to use the app.

## Available Scripts

| Command | What it does |
|---|---|
| `npm run dev` | Starts the Vite dev server with hot module reload |
| `npm run build` | Builds an optimized production bundle into `dist/` |
| `npm run preview` | Serves the production build locally, to sanity-check before deploying |
| `npm run lint` | Runs ESLint across the project |

## How to Use the App

1. Open the app — you'll see the **Input Parameters** card at the top.
2. Select one option each for **Gender**, **Age Group (yrs)**, and **Monthly Income (INR)**. A progress indicator shows how many of the 3 required inputs are selected.
3. Click **Generate Prediction**. The page scrolls down to the results.
4. Read through the four result cards:
   - Mode Choice Behavior
   - Public Transport Satisfaction
   - Gender-Based Travel Behavior
   - Scenario Responses & Suggestions
5. Note the **match banner** at the top of the results — it tells you exactly which demographic tier the shown data is based on, and the respondent count behind it.
6. (Optional) Click **Download PDF** to save the full result as a formatted PDF report.
7. Click **Reset** to clear the selections and try a different combination.

## How It Works

### 1. Data preparation (offline, one-time)

The raw survey (`data/tourist_survey.xlsx`, 107 responses) was aggregated outside the app into summary statistics per demographic segment — mean satisfaction scores, the most common categorical answers (e.g., primary mode, reason for choice), and respondent counts. These aggregates were converted into a nested JSON structure and embedded directly as the `LOOKUP` and `SUGGESTIONS` constants at the top of [`src/TouristModePredictor.jsx`](src/TouristModePredictor.jsx).

This means the app ships with all its "intelligence" baked in at build time — there's no API call, no database query, and no server involved when a prediction is generated at runtime.

### 2. The cascading lookup

A sample of 107 respondents can't densely cover every possible combination of 3 genders × 4 age groups × 4 income brackets (48 possible cells) — some cells have a single respondent, some have none. To handle this, the `lookup(gender, age, income)` function tries progressively broader keys until it finds data:

```
1. gender|age|income   (exact match)
2. gender|age
3. gender|income
4. age|income
5. gender
6. age
7. income
```

The first tier that has data wins, and the app reports both the tier name (`match`) and the respondent count (`n`) behind it — so the UI is always honest about how specific (or broad) the shown prediction really is.

### 3. Suggestions

Free-text improvement suggestions are pre-grouped by gender, age group, and income bracket (`SUGGESTIONS.by_gender`, `by_age`, `by_income`). `getSuggestions()` pools the suggestions relevant to all three selected values, deduplicates them, and returns up to 6.

### 4. Rendering

The results UI (`App` component in `TouristModePredictor.jsx`) renders four cards from the matched data object, using small presentational helpers:

- `StarRating` — converts a 1–5 score into a color-tiered bar (red/amber/green depending on score).
- `Badge` — renders yes/no/neutral pills for gender-safety and scenario answers.
- `formatMode` / `formatFreqMode` — shorten verbose survey category strings for display.

### 5. PDF export

`handleDownload` lazily imports [`src/generateReport.js`](src/generateReport.js) (code-split, so `jsPDF` isn't loaded unless a user actually downloads a report). That module manually draws a dark-themed report — header band, section headers, two-column data rows, satisfaction bars, scenario badges, and a suggestions list — directly onto a `jsPDF` canvas and triggers a client-side file save.

## Project Structure

```
tourist-mode-prediction/
├── data/
│   └── tourist_survey.xlsx       # Source survey data (107 responses)
├── public/
│   ├── favicon.svg
│   └── icons.svg
├── src/
│   ├── assets/                   # Static images (hero image, framework logos)
│   ├── App.jsx                   # Root component — mounts the predictor
│   ├── App.css
│   ├── TouristModePredictor.jsx  # LOOKUP/SUGGESTIONS data, matching logic, main UI
│   ├── TouristModePredictor.css  # Predictor-specific styling
│   ├── generateReport.js         # jsPDF-based PDF report builder
│   ├── index.css                 # Global styles
│   └── main.jsx                  # Vite/React entry point
├── index.html
├── vite.config.js
├── eslint.config.js
├── package.json
└── README.md
```

## Dataset & Variables

- **File:** `data/tourist_survey.xlsx` (single-sheet Google Forms export)
- **Sample size:** 107 tourist respondents
- **Timestamp column:** each row carries a form-submission timestamp (standard Google Forms behaviour); it is not used by the app and is not part of the aggregated `LOOKUP`/`SUGGESTIONS` data.

| Section | Variable | Type / Response options |
|---|---|---|
| A — Sociodemographic | Gender | Male / Female / Others |
| A | Age Group (yrs) | 18–25 / 26–35 / 36–45 / 46–60 |
| A | Marital Status | Single / Married |
| A | Education | Undergraduate / Graduate / Post Graduate / Diploma / Doctorate |
| A | Occupation | Student / Employed / Self Employed / Aspiring Employee / Homemaker |
| A | Monthly Income (INR) | <25k / 25k–50k / 50k–1L / >1L |
| A | Country | Indian / Foreigner |
| B — Trip Characteristics | Purpose of Visit | Leisure / Business / Education / Religious / Visiting Friends-Relatives |
| B | Accommodation Used | Hotel / Hostel / Guesthouse / Airbnb / Friends-Relatives |
| B | Duration of Stay | 1 day / 2–3 days / 4–7 days / >7 days |
| B | Travel Companion | Alone / Family / Friends / Tour Group |
| B | No. of Attractions Visited per Day | 1–2 / 3–4 / >4 |
| C — Mode Choice Behavior | Primary Mode Used within City | City Bus, Rental Cab, Auto-Rickshaw, Shared Taxi, Two-Wheeler, Private Car, Walk/Cycle (multi-select) |
| C | Most Frequently Used Mode | Public Transport / Private Transport / Mixed Transport |
| C | Main Mode Used for Arrival | Train / Bus / Flight / Private/Rented Vehicle |
| C | Reason for Mode Choice | Cost, Travel Time, Comfort, Safety, Availability, Gender Related Safety (multi-select) |
| C | Travel Time per Trip | <15 min / 15–30 min / 30–45 min / >45 min |
| C | Travel Cost per Trip (INR) | <50 / 50–100 / 100–150 / >150 |
| D — PT Usage & Satisfaction | Do you use Public Transport? | Regularly / Occasionally / Not Prefers |
| D | Satisfaction ratings (1–5 scale) | Availability, Affordability, Comfort, Safety, Accessibility of stops, Cleanliness, Driver/Conductor behaviour, Language understanding, Overall satisfaction |
| D | Scenario responses (would-you-use-PT-if…) | 8 items — single-route connectivity, ₹100–150 day pass, +50% frequency, fares +20% for comfort, live-tracking app, stops 5 min away, clear tourist info, eco-friendly promotion — each Yes / No / Not Sure |
| E — Gender-Based Travel Behavior | Does gender influence your travel mode choice? | Yes / No / May be |
| E | Do you feel public transport is safe for your gender? | Very Safe / Safe / Neutral / Unsafe |
| F — Suggestions | Suggestions to improve public transport for tourists | Free text (open-ended) |

The aggregated version of this data — grouped by demographic combination, with means computed for the rating fields and the most common categorical answer taken per group — is what actually powers the app at runtime; see [How It Works](#how-it-works).

## Data Anonymisation & Ethics

- The questionnaire did **not** collect any directly identifying information — no name, email address, phone number, or home address field exists anywhere in the instrument (verified by inspecting every column header in `tourist_survey.xlsx`).
- The only non-response field present is a Google Forms submission **timestamp**, which is not linked to any personal identifier and is excluded from the aggregated data embedded in the app.
- Free-text suggestion responses (Section F) were reviewed and contain general opinions about transport infrastructure; none reference a respondent's own name or other identifying detail.
- Respondents are therefore not identifiable from the released dataset, and no further de-identification step (e.g., redaction or pseudonymisation) was required beyond excluding the timestamp column from the app's aggregated data.

*(If the original raw survey export — as opposed to this app's pre-aggregated summary — is what gets shared publicly or archived, re-confirm the above against that exact file before release.)*

## Data & Code Availability Statement

The following is a ready-to-adapt statement for a paper/thesis referencing this project:

> The tourist mode-choice prediction tool described in this work is implemented as a client-side web application, deployed as a web application, and is publicly available through the project repository (https://github.com/alok-nitb/tourist-mode-prediction), distributed under the MIT License (see `LICENSE`). A snapshot of the repository has been archived on Zenodo and is assigned the DOI **10.5281/zenodo.23067833** (https://doi.org/10.5281/zenodo.23067833). Installation and execution instructions are provided in the repository's `README.md`. The underlying survey data (n = 107 tourist respondents) collected no directly identifying information; the only non-substantive field was an anonymous form-submission timestamp, which is excluded from the aggregated dataset embedded in the application. No separate MNL (multinomial logit) estimation script is included in this repository; the application performs demographic-segmented lookup and descriptive aggregation of survey responses rather than parametric choice modelling. Any MNL utility equations or probabilities reported elsewhere in this work were computed using a separate analysis not part of this codebase, and the exact software/package/version used for that analysis should be confirmed and cited independently of this repository.

## Limitations

- **Small, uneven sample:** some demographic combinations in the survey have very few respondents (as low as 1), so predictions for those cells — even after fallback — should be read as indicative, not statistically robust.
- **Static dataset:** the `LOOKUP`/`SUGGESTIONS` objects are hand-aggregated once and hardcoded in `TouristModePredictor.jsx`. There is currently no script in the repo to regenerate them automatically from an updated `tourist_survey.xlsx`.
- **Descriptive, not predictive modeling:** this is a segmented lookup of survey averages, not a trained statistical/ML model — there's no accuracy metric, cross-validation, or confidence interval attached to a "prediction."
- **No persistence:** selections and results are not saved between sessions; refreshing the page resets the app.

## Possible Improvements

- Add a data-processing script (e.g., a small Node or Python script) that regenerates `LOOKUP`/`SUGGESTIONS` directly from the Excel file, instead of maintaining that JSON by hand.
- Add basic automated tests for the `lookup()` fallback logic and the PDF generation pipeline.
- Add a chart-based view (e.g., comparing satisfaction across all segments at once) for a more analytical, less single-query experience.
- Grow the survey sample to better populate sparse demographic cells.
- Deploy the built app (e.g., to Vercel/Netlify) and link the live URL here.

## Notes for Future Me / Contributors

- Updating survey data means updating the `LOOKUP` and `SUGGESTIONS` constants near the top of `TouristModePredictor.jsx` — there is no separate data file loaded at runtime.
- The PDF layout in `generateReport.js` is built with raw jsPDF drawing primitives (rects, lines, text) rather than an HTML-to-PDF library, so any layout change there needs manual coordinate adjustments.
- No environment variables, API keys, or external services are required to run or build this project.

## License

This project is licensed under the **MIT License** — see [`LICENSE`](LICENSE) for the full text. When citing this repository's licence in a paper or thesis, state "MIT License" to match.
