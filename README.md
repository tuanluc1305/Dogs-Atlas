# 🐾 Dogs Atlas — U.S. Breed Explorer

> *Which dog breed dominates your state — and how has that changed over 13 years?*

**Dogs Atlas** is an interactive U.S. choropleth atlas built entirely as a **single self-contained HTML file** — no server, no build step, no npm install. Open it in any browser and start exploring.

![Dogs Atlas Preview](screenshots/dogs_atlas_preview.png)

---

## 🔗 Live Demo

Download `dogs-atlas.html` and open it directly in Chrome, Firefox, Safari, or Edge. No localhost needed.

---

## 🗺️ What It Shows

Two data views, one map:

**Dog Ownership Rate** — A heatmap of estimated household dog ownership across all 50 states + DC, sourced from AVMA regional surveys. Wyoming leads at 67%; DC sits at 23%.

**Breed Popularity by State** — Select any of 202 AKC-recognized breeds and see where it ranks in each state. The color scale runs from white (not in top 10) to the darkest palette tone (ranked #1 in that state).

---

## ✨ Features

| Feature | Detail |
|---|---|
| 🗺️ U.S. Choropleth Map | Accurate state boundaries from Wikimedia SVG reference, 51 states + DC |
| 🌡️ Heatmap | Percentile-capped color scale (p2–p98) prevents outliers from washing out the gradient |
| 🎨 5 Color Palettes | Cool Blue · Heat · Teal · Amber · Violet — all update map, legend, charts, and sparklines simultaneously |
| 🔍 Zoom & Pan | Scroll to zoom (up to 8×), drag to pan, pinch on touch, zoom-out below fit to see Hawaii/Alaska |
| 📅 Year Slider | Scrub through AKC national rankings from **2013 to 2025** (13 years) |
| 🔎 Live Search | Instant search across 202 breeds and 51 states |
| 📊 Trend Chart | 13-year line chart of any breed's national rank over time |
| 📋 National Rankings Table | 200+ breeds with sparklines, Δ1yr, Δ10yr columns, group filter |
| 🐕 Breed Photos | Canonical breed photos via Wikipedia REST API (Wikimedia Commons, CC-licensed) |
| 📍 State Panel | Top breeds per state, ownership rate bar, data source badge |
| 🏅 Breed Panel | AKC group, size, lifespan, national rank, state appearances |

---

## 📸 Screenshots

### Dog Ownership Rate — Amber Palette
![Ownership Rate](screenshots/ss_ownership.png)

### Selected Breed Popularity — Breed Mode
![Breed Mode](screenshots/ss_breed_heat.png)

---

## 📦 Repository Structure

```
dogs-atlas/
│
├── dogs-atlas.html              ← The entire application (open in any browser)
├── README.md                    ← This file
├── DATA_SOURCES.md              ← Full sourcing, methodology, and transparency notes
│
├── screenshots/
│   ├── dogs_atlas_preview.png   ← Project profile photo
│   ├── ss_ownership.png         ← Ownership rate heatmap
│   └── ss_breed_heat.png        ← Breed popularity mode
│
└── data/
    └── AKC_Dog_Breed_Ranking_2013_2023.xlsx   ← Raw source data (Kaggle)
```

---

## 🔢 Data Coverage

| Dimension | Coverage |
|---|---|
| Years | 2013 – 2025 (13 years) |
| Unique breeds tracked | 205 across all years |
| Breeds in 2023 | 200 |
| Breeds in 2024–2025 | 165 (partial — AKC publishes full lists gradually) |
| AKC breed groups | All 7 official groups |
| States with city-derived data | NY, IL, WA |
| States with estimated data | 48 states + DC |

See [`DATA_SOURCES.md`](DATA_SOURCES.md) for full sourcing, field-by-field methodology, and transparency notes.

---

## 🛠️ How It's Built

Zero external frameworks. Zero build pipeline. Everything runs in the browser.

| Layer | Technology |
|---|---|
| Map rendering | Pre-projected inline SVG paths (no D3 at runtime, no TopoJSON fetch) |
| Charts & sparklines | [Chart.js 4](https://www.chartjs.org/) via CDN |
| Color interpolation | Custom 6-stop palette lerp (vanilla JS) |
| Breed photos | [Wikipedia REST API](https://en.wikipedia.org/api/rest_v1/) → Wikimedia Commons |
| Fonts | [Google Fonts](https://fonts.google.com/) — Playfair Display + Inter |
| Styling | Vanilla CSS with custom properties |
| State boundaries | Wikimedia Commons blank U.S. SVG (public domain) |
| Data | Embedded JS objects — no external JSON fetches |

The map is fully self-contained: state SVG paths were pre-projected at build time using D3's `geoAlbersUsa` projection (975×610 coordinate space) and embedded directly in the HTML. This means the map renders instantly with zero network requests for map data.

---

## 🧪 Data Processing

The raw AKC ranking data (`AKC_Dog_Breed_Ranking_2013_2023.xlsx`) was processed using Python + pandas:

```python
import pandas as pd, json

df = pd.read_excel('data/AKC_Dog_Breed_Ranking_2013_2023.xlsx')

# Year-keyed ranking dictionary
akc_rankings = {}
for year in range(2013, 2024):
    col = f'Ranking {year}'
    akc_rankings[str(year)] = {
        row['Breed']: int(row[col])
        for _, row in df.iterrows()
        if pd.notna(row[col])
    }

# Breed → AKC group mapping
breed_groups = {
    row['Breed']: row['Group'].replace(' Group', '')
    for _, row in df.iterrows()
}
```

2024 and 2025 rankings were transcribed from the official AKC annual announcements and merged into the same structure.

---

## ⚠️ Known Limitations

- **No official state-level breed data exists in the U.S.** AKC registrations are national only. State-level breed rankings are proxied from city licensing datasets (NY, IL, WA) or estimated from AVMA regional surveys. All data labels are surfaced in the UI.
- **2024–2025 rankings are partial.** AKC publishes ~165 of ~202 breeds in their public announcements.
- **AKC rankings reflect registrations, not total population.** Mixed breeds and unregistered dogs are not captured.

---

## 🗺️ Credits

| Resource | Source |
|---|---|
| AKC breed rankings 2013–2023 | [Kaggle dataset](https://www.kaggle.com/) + American Kennel Club |
| AKC breed rankings 2024–2025 | [AKC 2024](https://www.akc.org/expert-advice/news/most-popular-dog-breeds-2024/) · [AKC 2025](https://www.akc.org/expert-advice/dog-breeds/most-popular-dog-breeds-2025/) |
| Dog ownership by state | [AVMA Pet Ownership Statistics](https://www.avma.org/resources-tools/reports-statistics/us-pet-ownership-statistics) |
| City licensing data | [NYC Open Data](https://data.cityofnewyork.us) · [Seattle Open Data](https://data.seattle.gov) · [Chicago Data Portal](https://data.cityofchicago.org) |
| Breed photos | [Wikipedia REST API](https://en.wikipedia.org/api/rest_v1/) / Wikimedia Commons |
| U.S. state boundaries | Wikimedia Commons (public domain SVG) |

---

## 🚀 Future Ideas

- [ ] Shelter/rescue intake data layer (Petfinder API)
- [ ] Doodles + hybrid breed category
- [ ] County-level data for states with open licensing data
- [ ] Time-lapse animation mode
- [ ] Shareable URL state (encode breed/state/year in URL hash)
- [ ] Breed comparison mode

---

## 📄 License

MIT — fork it, adapt it, build on it.

---

*Developed by Luc Lu*
