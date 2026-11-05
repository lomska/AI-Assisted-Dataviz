# Data dictionary — `data/transformed/tour_ce_omn12_long.csv` and `data/ready/tour_ce_omn12_indices.csv`

Nights spent in short-stay accommodation booked through online platforms (Airbnb, Booking, Expedia, Tripadvisor), by month, guest residence and region, **2018–2025**.

| File | Made by | Content |
|---|---|---|
| `data/transformed/tour_ce_omn12_long.csv` | `Stage 1 Transform.ipynb` | **Clean, tested data from the raw Eurostat files**: identifiers, nights (Value) and the regions' geography shares — 14 columns, 118,794 rows |
| `data/ready/tour_ce_omn12_indices.csv` | `Stage 2 Indices.ipynb` | The same rows **plus all indices** (37 columns) — use this one for maps and charts |

Three notebooks, run in order: the **transform** notebook processes all raw files and tests the result against them; the **indices** notebook (`Stage 2 Indices.ipynb`) reads only the clean CSV, adds and explains the indices (A1–A10, robustness checks in its appendix R1–R4) and saves the indices file; the **findings** notebook (`Stage 3 Analysis.ipynb`) reads the indices file and answers the research questions, with a summary, conclusions and a check of every number quoted in its text. Transform needs pandas, numpy and openpyxl; indices and findings need pandas, numpy, scipy and matplotlib.

## Sources (Eurostat)

| File | Content |
|---|---|
| `data/raw/tour_ce_omn12__custom_23247212_spreadsheet.xlsx` | *tour_ce_omn12*, nights spent, extract of 15/10/2026 (data updated 02/10/2026) |
| `data/raw/NUTS2021-NUTS2024.xlsx` | NUTS 2024 codes and labels; NUTS 3 mountain, island and coastal typologies |
| `data/raw/EU-27-LAU-2024-NUTS-2024.xlsx` | Municipalities (LAU 2024): NUTS 3 code, population, degree of urbanisation, coastal flag |

## Methods literature

* Fernández-Morales, A., Cisneros-Martínez, J. D., & McCabe, S. (2016). Seasonal concentration of tourism demand: Decomposition analysis and marketing implications. *Tourism Management*, 56, 172–190. (Gini index, decomposition, RME, classes)
* Lo Magno, G. L., Ferrante, M., & De Cantis, S. (2017). A new index for measuring seasonality: A transportation cost approach. *Mathematical Social Sciences*, 88, 55–65. (S_R; implementation checked against their Table 2)
* Ferrante, M., Lo Magno, G. L., & De Cantis, S. (2018). Measuring tourism seasonality across European countries. *Tourism Management*, 68, 220–235. (clustering of seasonal profiles; used to check SeasonPattern)
* Zhang, L., & Qian, Y. (2003). Annual distribution features of precipitation in China and their interannual variations. *Acta Meteorologica Sinica*, 17(2), 146–163. (PCD and the centre of the year, as discussed by Lo Magno et al. 2017)

## File format

* CSV, comma-separated, UTF-8 with BOM (so Excel shows accented names correctly); empty field = no value.
* 118,794 rows in both files; 14 columns in the clean file, 37 in the indices file.
* **One row = one Region × NUTSLevel × Country × Year × Month × ResidenceCountry.** Region names alone are not unique (*Berlin*, *Malta*, *Luxembourg* appear at several levels); use NUTSCode, or Region together with NUTSLevel and Country.
* Coverage: EU-27 plus Iceland, Liechtenstein, Norway, Switzerland — 31 countries, 97 NUTS 1 and 260 NUTS 2 regions, NUTS 2024 version.

**Typical filters**

| Purpose | Filter |
|---|---|
| Annual map of regions | `Month == "Total"`, `ResidenceCountry == "Total"`, one `Year`, one `NUTSLevel` |
| Bivariate map (foreign share × Gini) | as above; colour by `BivariateCode`, outline where `SeasonShape == "Two seasons"`, join to boundaries on `NUTSCode` |
| Monthly profile of one region | `Month != "Total"`, one `NUTSCode`, `Year`, `ResidenceCountry` |
| Country comparison | `NUTSLevel == 0` |

## Columns

The **Identifiers**, **Value** and the geography **shares** (PrcCities, PrcRural, PrcCoastal, PrcInlandMountain, PrcIsland) are in both files; everything else — all indices and the geography groups — only in `data/ready/tour_ce_omn12_indices.csv`.

### Identifiers

| Column | Type | Description |
|---|---|---|
| Region | text | Country name (level 0) or NUTS region name as published by Eurostat |
| NUTSCode | text | NUTS 2024 code (e.g. `DE11`); country code for level 0 (`EL` = Greece) |
| NUTSLevel | integer | 0 = country, 1 = NUTS 1, 2 = NUTS 2 |
| ParentRegion | text | NUTS 2 → its NUTS 1 region; NUTS 1 → its country; **empty** for countries |
| Country | text | Country the region belongs to |
| Year | integer | 2018–2025 |
| Month | text | `January` … `December`, or `Total` (annual total; use `Month == "Total"` for annual rows) |
| ResidenceCountry | text | `Domestic`, `Foreign`, or `Total` (Eurostat's published total over all residences) |

### Nights and seasonality

| Column | Type | Description | Empty when |
|---|---|---|---|
| Value | integer | Nights spent | never |
| CumPercentage | number (2 dp) | Running share of the annual total (%) after ordering the year's months by Value, largest first (ties in calendar order) | Total rows; Latvija 2024 Foreign/Total (annual total ≠ sum of months) |
| Prc75Flag | 0/1 | 1 for the months up to and including the first one where CumPercentage ≥ 75 (the crossing month), plus any month with exactly the same Value as the crossing month | Total rows; Latvija 2024 Foreign/Total (annual total ≠ sum of months) |
| ThresholdMonth | 0/1 | 1 for months tied in Value with the crossing month (2+ identical months at the 75% break), else 0 | Total rows; Latvija 2024 Foreign/Total (annual total ≠ sum of months) |
| Season75 | integer | Number of months with Prc75Flag = 1 in the region-year-residence: the peak months needed to reach 75% of the year (smaller = more seasonal; an even year needs 9) | Total rows; Latvija 2024 Foreign/Total (annual total ≠ sum of months) |

### Seasonality indices (one value per region-year-residence, on all its rows)

| Column | Type | Description | Empty when |
|---|---|---|---|
| VolumeOK | 0/1 | 1 if the year has at least 10,000 nights (and all 12 months, and a published annual total equal to the sum of its months); the indices below are only given for VolumeOK = 1 | never |
| Gini | number (4 dp) | Gini index of the 12 monthly values (covariance form, Fernández-Morales et al. 2016): 0 = all months equal, 0.917 = everything in one month; ignores the order of months | VolumeOK = 0 |
| SeasonalityPercentile | integer | Gini on a 0–100 scale: % of comparable units with a lower Gini (same NUTS level, year and ResidenceCountry; VolumeOK series only; Extra-Regio excluded). 0 = least, 100 = most seasonal | VolumeOK = 0 |
| GiniMapGroup | text | Gini in three roughly equal groups for maps: `Low` (< 0.20), `Medium` (0.20 to < 0.30), `High` (≥ 0.30); the 2025 NUTS 2 tertiles are 0.195 / 0.307 | VolumeOK = 0 |
| SeasonShape | text | `No clear season` (Gini < 0.15, the low level of Fernández-Morales et al. 2016), `One season` or `Two seasons` (separate peaks, e.g. winter + summer). One vs. two is decided by comparing the circular concentration PCD (Zhang & Qian 2003) with S_R (Lo Magno et al. 2017): PCD ÷ S_R ≥ 0.8 = one season. Both are computed in the indices notebook but not stored. Rule of this dataset, not from the literature. Outline `Two seasons` regions on a Gini map | VolumeOK = 0 |
| SeasonCentre | number (2 dp) | Centre of the year's nights on the year circle as a month position where each month sits at its whole number: 1.0 = January, 7.0 = July, 12.0 = December, so 7.5 = halfway between July and August and 12.5 = between December and January (cyclic, values 1.0–12.99) | not "One season" |
| SeasonCentreMonth | text | Month nearest to SeasonCentre | not "One season" |
| SeasonPattern | text | Shape of the year, five patterns, first matching rule: `Mild or no season` (no clear season, Gini < 0.15); `Winter-based seasons` (two seasons, or one season centred Nov–Mar); `Summer season + winter shoulder` (summer season, Dec–Feb at least an average month); `Strong summer season` (Jul–Aug ≥ 2.5 × average month); `Summer season` (1.5–2.5); `Mild or no season` (< 1.5). Per series, like Gini. Robustness tested in the indices notebook (appendix R3–R4): Strong summer, Summer, Winter-based and Mild or no season keep their pattern from year to year in 87–98% of NUTS 2 regions and agree with a Ward clustering of the profiles (Ferrante et al. 2018) for 83% of regions; *Summer season + winter shoulder* is small (7 NUTS 2 regions in 2025) and sensitive to its winter cut-off — read as "summer regions with some winter business". The patterns hold about as well for foreign guests alone; for domestic guests Winter-based and Summer + winter shoulder blur | VolumeOK = 0 |

### Foreign share

| Column | Type | Description |
|---|---|---|
| PrcForeign | number (2 dp) | Foreign ÷ Total nights × 100 for the region, year and month; the same value on the Domestic, Foreign and Total rows. Empty for the 4 Mount Athos months with no nights at all (0 ÷ 0) and where Eurostat's Total differs from Domestic + Foreign by more than 1% (only Latvija 2024: October–December and the annual Total). With `:` replaced by 0 the earlier rules (Foreign missing → 0, Domestic missing → 100) no longer occur |
| ForeignGroup | text | `Mainly domestic` (< 40%), `Mixed` (40 to < 70%), `Mainly foreign` (≥ 70%) |
| BivariateCode | two digits | Class for a 3 × 3 map, ForeignGroup × GiniMapGroup: foreign group first (1 mainly domestic, 2 mixed, 3 mainly foreign), Gini group second (1 low, 2 medium, 3 high), e.g. `33` = mainly foreign × high. **Annual rows of all guests only** (`Month == "Total"`, `ResidenceCountry == "Total"`): the region's foreign share with the Gini index of all guests. Empty where VolumeOK = 0 |
| RMEForeign | number (4 dp) | Relative marginal effect of foreign guests on the region's Gini index (Fernández-Morales et al. 2016; Lerman & Yitzhaki 1985): relative change of the Gini index if foreign demand grew slightly. Negative = foreign guests reduce seasonality. The domestic value is −RMEForeign. Empty where Domestic + Foreign < 10,000 nights and for Latvija 2024 (broken source series) |
| ForeignSeasonEffect | text | `Smoothing` (RMEForeign ≤ −0.02: foreign guests smooth the year), `Small or unclear` (between −0.02 and 0.02: the band is about as wide as a region's normal year-to-year movement of RMEForeign, so this means "cannot be told apart from zero", not "zero"), `Sharpening` (≥ 0.02: they sharpen the peak) — classes of Fernández-Morales et al. (2016), called favourable / somewhat favourable / unfavourable there |
| ForeignTimingEffect | number (4 dp) | How much of the region's seasonality lies in foreign guests' timing, in Gini points: Gini of all nights − Gini if foreign guests kept their number of nights but spread them over the year like the region's domestic guests. Positive = foreign timing makes the year more seasonal, negative = less seasonal, 0 = foreign guests travel like domestic guests. An accounting comparison, not a causal effect. One value per region-year, on all its rows. Empty unless both guest groups have all 12 months and at least 10,000 nights (and for Latvija 2024). Stable from year to year (rank correlation 0.96, 2024→2025) |
| ForeignTimingClass | text | `Adds` (ForeignTimingEffect > +0.03), `No clear effect` (−0.03 to +0.03), `Reduces` (< −0.03); 0.03 is the upper end of the normal year-to-year movement of the Gini index. NUTS 2, 2025: Adds 105, No clear effect 122, Reduces 27 regions |

### Geography (residents, 2024; same values for every year)

| Column | Type | Description |
|---|---|---|
| PrcCities / PrcRural | number (2 dp) | % of the region's residents living in cities / rural areas (DEGURBA classes of their municipality); towns and suburbs = 100 − PrcCities − PrcRural |
| UrbanGroup | text | `Predominantly urban` (PrcRural ≤ 20%), `Intermediate`, `Predominantly rural` (PrcRural ≥ 50%) |
| PrcCoastal | number (2 dp) | % of residents in coastal municipalities |
| CoastalGroup | text | `Non-coastal` (0%), `Partly coastal` (< 50%), `Predominantly coastal` (≥ 50%) |
| PrcInlandMountain | number (2 dp) | % of residents in NUTS 3 regions that are mountain **and** non-coastal |
| InlandMountainGroup | text | `Non-inland mountain`, `Partly inland mountain`, `Predominantly inland mountain` (same cuts) |
| PrcIsland | number (2 dp) | % of residents in NUTS 3 regions classed as island |
| IslandGroup | text | `Non-island`, `Partly island`, `Predominantly island` (same cuts) |

Geography columns are empty for **Iceland** (not in the LAU file) and, for mountain and island only, for **Liechtenstein** and Greece's **Extra-Regio** area (Mount Athos), which the NUTS 3 typology sheet does not cover.

## Processing notes and caveats

* **Computed but not stored** (intermediate steps, in the indices notebook): the running sum behind CumPercentage, S_R and PCD (used for SeasonShape), the Gini correlation of foreign guests (used for RMEForeign). The towns-and-suburbs share is 100 − PrcCities − PrcRural.
* **ForeignSeasonEffect** is classified from the stored (4-decimal) RMEForeign, so values of exactly ±0.0200 fall in *Smoothing* / *Sharpening*.

* **`:` (not available) → 0** inside series that are otherwise published (51 cells in 31 series: Liechtenstein domestic, Ceuta and Melilla foreign, Mount Athos 2025); in all of them the published months add up exactly to the published annual total. Series with no published value at all for a year are **left out**, not filled: the Dutch and Portuguese NUTS 2024 regions before 2024, Greece's Extra-Regio area 2018–2024, and Iceland's region *Ísland* in 2025 (the Iceland country row has 2025).
* **Removed:** **2026** (only January–March published, no annual total); the superseded **NUTS 2021** regions of the Netherlands and Portugal. Their NUTS 2024 replacements (Utrecht, Zuid-Holland; Centro (PT), Grande Lisboa, Península de Setúbal, Alentejo, Oeste e Vale do Tejo) start in 2024, so those NUTS 2 regions are missing for 2018–2023 and are left out of any multi-year regional comparison.
* **Season75 threshold:** 75% was tested against 50–90%. Thresholds from 60% to 75% agree best with the Gini index (rank correlation 0.96–0.97) and change between consecutive years in 16–19% of series; above 80% the agreement drops (0.92 at 90%) and changes become more frequent (about 20%). Season75 is kept as the easy-to-explain companion of the Gini index ("three-quarters of the nights fall in N months") and Prc75Flag shows which months those are; use Gini / GiniMapGroup for classification.
* **Minimum-volume rule:** series with fewer than 10,000 nights a year (112 of 9,138, e.g. Liechtenstein domestic, Ceuta, Melilla, Mayotte, Mount Athos, and the broken Latvija 2024 Foreign/Total) have VolumeOK = 0: no Gini, shape, centre, season pattern or map groups. Season75 and the month flags are still given, except for the broken Latvija 2024 Foreign/Total series. Below 10,000 nights the year-to-year change of S_R and Gini is 3–5 times that of larger series. The Gini decomposition needs Domestic and Foreign with 12 months each and 10,000 nights together.
* **Corrected:** in one-region countries a regional value above the country value is replaced by the country value. This changed 6 monthly cells of *Ísland* (Iceland, NUTS 1 and 2, 2024: Oct Foreign, Nov and Dec Domestic); its 2024 Domestic and Foreign annual totals were recomputed as the sum of the months. All other values are as published.
* **Known source error, kept as published:** *Latvija* 2024 (NUTS 1 and 2), Foreign and Total: annual total 268 and Oct–Dec values near zero. Use the Latvia country row for 2024. In the indices file these two series carry no indices, Latvija 2024 has no RMEForeign, and PrcForeign is empty for its October–December and annual rows.
* **Country ≠ sum of regions:** country totals can include nights not allocated to any region (notably Norway 2024, gaps of up to 49%), so for country figures use the NUTSLevel 0 rows rather than summing regions.
* **PrcForeign** is computed after `:` → 0, so a missing foreign or domestic month counts as 0 nights.
* **Geography shares** describe where residents live, not where guests stay. The urban–rural, coastal and 50% cuts mirror Eurostat's NUTS 3 rules but are applied here to NUTS 1/2 and countries, which Eurostat does not officially classify. Eurostat's mountain definition is broad (most Mediterranean islands and hinterland count as mountain); PrcInlandMountain is the closer measure of mountain destinations, though some Swiss and Austrian lowland areas still count (e.g. Zürich 100%).
* Coverage is limited to stays booked through the four online platforms; growth, and part of the fall in seasonality since 2019, partly reflect wider platform use and a changing mix of users, not only changes in travel.
* **Month length** is not adjusted for: February's fewer days raise the Gini index slightly (EU-27, 2025: 0.261 as published, 0.257 per day).
* **Column renamed** on 2 November 2026: `TresholdMonth` → `ThresholdMonth`.
* **Which foreign-guest measure to use:** for "how much do foreign guests shape the season today" use **ForeignTimingEffect / ForeignTimingClass** (added 2 November 2026); RMEForeign / ForeignSeasonEffect answer the narrower question of what a small increase in foreign demand would do; Gini foreign vs Gini domestic only says which group is more seasonal.
* **ForeignSeasonEffect** class `Neutral` renamed `Small or unclear` on 2 November 2026. Sharpening and Smoothing keep their class through 2023–2025 in about 80% of NUTS 2 regions. RMEForeign is a *marginal* effect: 10% more foreign nights would move a typical region's Gini by only about 0.002. Where foreign guests are > 85% of nights, RMEForeign is mechanically close to 0. How much of the seasonality lies in foreign guests' timing (a counterfactual) is ForeignTimingEffect, the main measure of the findings notebook (section 3.3).
* **SeasonPattern reduced from seven to five patterns** on 2 November 2026: *Year-round* and *Mild summer season* → `Mild or no season`; *Winter and summer seasons* and *Winter season* → `Winter-based seasons`. The two merged pairs were not stable or not distinct enough to report separately.
* **Reading Gini differences:** a region's Gini typically moves about 0.01 between years (90% of moves below about 0.025), so differences under 0.02–0.03 are within normal movement. A few regions sit near the one/two-season border (PCD ÷ S_R 0.7–0.9) and switch SeasonShape between years, e.g. Tirol, Kärnten, Innlandet, the Norrland regions.
