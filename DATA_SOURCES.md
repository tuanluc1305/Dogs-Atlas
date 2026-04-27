# 📊 Data Sources & Methodology

This document explains every data source used in Dogs Atlas, how each was obtained, how it was transformed, and what its limitations are. Every metric shown in the app has a corresponding entry here.

All data fields in the app are labeled with one of four badges:

| Badge | Meaning |
|---|---|
| ✅ **Direct** | Taken directly from a primary public source without transformation |
| 📊 **Derived** | Calculated or aggregated from a primary source |
| `〜` **Estimated** | Extrapolated from regional data or proxy sources |
| 📌 **Reference** | Used as metadata/context, not as a plotted metric |

---

## 1. AKC National Breed Rankings (2013–2025)

**What it is:** Annual national popularity ranking of AKC-recognized dog breeds by total new registrations in the U.S. for each calendar year.

**Badge:** ✅ Direct

**Sources:**
- **2013–2023:** `AKC_Dog_Breed_Ranking_2013_2023.xlsx` — a structured dataset published on [Kaggle](https://www.kaggle.com/datasets/marshuu/dog-breeds), derived from AKC's official annual announcements. The Excel file contains one row per breed and one column per year (`Ranking 2013` through `Ranking 2023`), with the breed group also included.
- **2024:** [AKC Most Popular Dog Breeds 2024](https://www.akc.org/expert-advice/news/most-popular-dog-breeds-2024/) — official AKC announcement. Top ~165 breeds transcribed from the published table.
- **2025:** [AKC Most Popular Dog Breeds 2025](https://www.akc.org/expert-advice/dog-breeds/most-popular-dog-breeds-2025/) — official AKC announcement. Top ~165 breeds transcribed from the published table.

**How it was processed (2013–2023):**
```python
import pandas as pd

df = pd.read_excel('data/AKC_Dog_Breed_Ranking_2013_2023.xlsx')

akc_rankings = {}
for year in range(2013, 2024):
    col = f'Ranking {year}'
    akc_rankings[str(year)] = {
        row['Breed']: int(row[col])
        for _, row in df.iterrows()
        if pd.notna(row[col])
    }

breed_groups = {
    row['Breed']: row['Group'].replace(' Group', '')
    for _, row in df.iterrows()
}
```

**How it was processed (2024–2025):**  
Manually transcribed from the AKC announcement pages into the same `{ "Breed Name": rank }` dictionary format and merged with the existing dataset.

**Coverage:**

| Year | Breeds Ranked |
|---|---|
| 2013 | 177 |
| 2014–2015 | 184 |
| 2016 | 189 |
| 2017 | 190 |
| 2018 | 192 |
| 2019–2020 | 195 |
| 2021 | 197 |
| 2022 | 199 |
| 2023 | 200 |
| 2024–2025 | 165 (partial) |

**Limitations:**
- AKC rankings are based on new registrations, not total dogs owned. Breeds popular among owners who don't register with AKC (especially mixed breeds and certain working dogs) are underrepresented.
- The 2024 and 2025 lists are partial because AKC typically publishes only the top portion of the full list in their public announcements. The complete ranked lists require access to AKC's full registration database.
- Rankings reflect the prior calendar year's registrations (e.g., the "2025" list reflects 2024 registration data).

---

## 2. AKC Breed Metadata

**What it is:** Breed group classification, size category, and lifespan for each breed.

**Badge:** ✅ Direct (group) · 📌 Reference (size, lifespan)

**Sources:**
- **Breed group:** Taken directly from the `Group` column in `AKC_Dog_Breed_Ranking_2013_2023.xlsx`. AKC's 7 official groups: Sporting, Hound, Working, Terrier, Toy, Non-Sporting, Herding.
- **Size and lifespan:** Compiled from AKC breed profile pages (`akc.org/dog-breeds/[breed-name]/`) and cross-referenced with the Kaggle dataset. These are reference values and not plotted as metrics in the map.

**Breed group counts in dataset:**

| Group | Breeds |
|---|---|
| Sporting | 33 |
| Herding | 32 |
| Hound | 32 |
| Working | 31 |
| Terrier | 31 |
| Toy | 21 |
| Non-Sporting | 20 |

---

## 3. State-Level Breed Popularity

**What it is:** A ranked list of the top 10 most popular breeds in each U.S. state.

> ⚠️ **Important:** No official state-level dog breed registration database exists in the United States. AKC registration data is published at the national level only. The state-level breed data in this app is the closest available public proxy, with transparent labeling.

### 3a. Derived States — City Licensing Data

**Badge:** 📊 Derived

**States:** New York (NY), Illinois (IL), Washington (WA)

These states have large cities that publish open dog licensing datasets. Breed rankings were derived from counting licensed dogs by breed within each city dataset:

| State | City Dataset | Source URL |
|---|---|---|
| New York | NYC Dog Licensing Dataset | [data.cityofnewyork.us](https://data.cityofnewyork.us/Health/NYC-Dog-Licensing-Dataset/nu7n-tubp) |
| Illinois | Chicago Animal Care & Control | [data.cityofchicago.org](https://data.cityofchicago.org/Health-Human-Services/Chicago-Animal-Care-and-Control-Lost-Found-Adoptable-Pets/kqrt-9anm) |
| Washington | Seattle Pet Licenses | [data.seattle.gov](https://data.seattle.gov/Community/Seattle-Pet-Licenses/jguv-t9rb) |

**Caveat:** These rankings reflect the licensing behavior of one major city within each state, not the full state population. Urban licensing patterns may differ from rural or suburban areas of the same state.

### 3b. Estimated States — AVMA Regional Data

**Badge:** `〜` Estimated

**States:** All remaining 48 states + DC

For states without accessible city-level breed licensing data, state-level breed rankings are estimated using:
1. AVMA regional breed preference data (Northeast, South, Midwest, Southwest, West)
2. State-specific adjustments based on known demographic and geographic patterns (e.g., Alaska's husky prevalence, Hawaii's urban breed mix)
3. Cross-referencing with AKC national rankings to ensure regional patterns are plausible

**Source:** [AVMA Pet Ownership & Demographics Sourcebook](https://www.avma.org/resources-tools/reports-statistics/us-pet-ownership-statistics)

**Caveat:** These are regional estimates, not state-level measurements. They should be read as "states in this region tend to favor these breeds" rather than precise state-level data.

---

## 4. Dog Ownership Rate by State

**What it is:** Estimated percentage of households in each state that own at least one dog.

**Badge:** `〜` Estimated

**Source:** AVMA Pet Ownership & Demographics Sourcebook — [avma.org](https://www.avma.org/resources-tools/reports-statistics/us-pet-ownership-statistics)

**How it works:** The AVMA publishes household dog ownership rates broken down by U.S. region and selected states. Where state-level figures were available, they were used directly. For states not individually reported, the regional average was applied and adjusted based on available state-level indicators (urbanization rate, median income, housing type distribution).

**Range in dataset:**
- Lowest: DC at 23%
- Highest: Wyoming at 67%
- National median: ~40%

**Color scaling:** The heatmap uses percentile-capped scaling (p2–p98) so that DC (23%) and Wyoming (67%) don't compress the rest of the country into an undifferentiated middle range.

---

## 5. Breed Photos

**What it is:** A representative profile photo for each breed shown in the breed detail panel.

**Badge:** 📌 Reference

**Primary source:** [Wikipedia REST API](https://en.wikipedia.org/api/rest_v1/page/summary/{breed_name})

Each breed photo is fetched at runtime from Wikipedia's public REST API, which returns the canonical article summary including the primary image. These images come from [Wikimedia Commons](https://commons.wikimedia.org/) and are licensed under Creative Commons (CC BY-SA or similar) with attribution.

**Why Wikipedia instead of AKC:** AKC breed images are served from their CDN and not available under an open license for third-party use. Wikipedia/Wikimedia Commons images are explicitly licensed for reuse with attribution.

**Fallback for breeds with composite images:** A small number of breeds (Belgian Malinois, Poodle, Pointer) have multi-photo collages as their Wikipedia representative image. These breeds are routed to the [Dog CEO API](https://dog.ceo/dog-api/) as a fallback, which serves individual dog photos from the Stanford Dogs Dataset under an open license.

**Photo credit:** "© Wikimedia Commons" is shown on each photo in the app.

---

## 6. U.S. State Boundaries

**What it is:** SVG path data for all 50 states + DC used to render the choropleth map.

**Badge:** 📌 Reference

**Source:** [Wikimedia Commons — Blank US Map (states only)](https://commons.wikimedia.org/wiki/File:Blank_US_Map_(states_only).svg) — Public domain.

**Processing:** State paths were extracted from the SVG file by parsing each `<path class="[state-abbr]">` element. The paths use the SVG file's native 959×593 coordinate system, which already includes the standard AK/HI inset positioning. Paths were embedded directly in the HTML — no fetch, no D3 projection at runtime.

---

## Summary Table

| Metric | Source | Badge | Notes |
|---|---|---|---|
| National breed rank (2013–2023) | AKC via Kaggle | ✅ Direct | Full 177–200 breed lists |
| National breed rank (2024–2025) | AKC announcements | ✅ Direct | Partial (~165 breeds) |
| Breed group | AKC via Kaggle | ✅ Direct | 7 official AKC groups |
| Breed size / lifespan | AKC breed profiles | 📌 Reference | Display only, not plotted |
| State breed rankings (NY, IL, WA) | City licensing data | 📊 Derived | City-level proxy for state |
| State breed rankings (all others) | AVMA regional data | `〜` Estimated | Regional proxy, not state-level |
| Dog ownership rate | AVMA sourcebook | `〜` Estimated | Regional → state extrapolated |
| Breed photos | Wikipedia REST API | 📌 Reference | CC-licensed, runtime fetch |
| State map boundaries | Wikimedia Commons SVG | 📌 Reference | Public domain |
