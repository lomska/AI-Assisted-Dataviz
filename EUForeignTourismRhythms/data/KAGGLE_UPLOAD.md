# Kaggle upload — texts for each field

Two ways to upload:

* **Web form** (kaggle.com → Create → New Dataset): upload `data/ready/tour_ce_omn12_indices.csv`, then paste the texts below into the matching fields.
* **Command line**: edit the `id` in `data/ready/dataset-metadata.json` (replace `YOUR_KAGGLE_USERNAME`), then run `kaggle datasets create -p data/ready`. The file already contains the title, subtitle, description, license, tags and all column descriptions. Only the CSV and the metadata file are in that folder, so nothing else is uploaded.

## Title (6–50 characters; 48 used)

EU Platform Tourism: Seasonality Indices 2018-25

## Subtitle (20–80 characters; 78 used)

Monthly nights by guest origin in 31 countries and their regions, with indices

## URL slug

eu-platform-tourism-seasonality-indices

## License

CC BY 4.0 (Attribution 4.0 International)

## Tags

travel, europe, tourism, time series analysis, geospatial analysis, data visualization, economics, government

## Provenance

**Sources:** Eurostat, tour_ce_omn12 — nights spent in short-stay accommodation booked via online platforms, monthly (https://ec.europa.eu/eurostat/databrowser/view/tour_ce_omn12/default/table), extract of 15 October 2026; Eurostat NUTS 2024 classification and NUTS 3 typologies; Eurostat LAU 2024 municipalities.

**Collection methodology:** Downloaded from the Eurostat data browser; cleaned and tested against the raw files (`Stage 1 Transform.ipynb`); indices added, explained and robustness-checked (`Stage 2 Indices.ipynb`); findings in `Stage 3 Analysis.ipynb`. Eurostat's `:` inside published series set to 0; 2026 and NUTS 2021 regions removed; Ísland 2024 corrected in 6 cells. Eurostat bears no responsibility for these modifications.

## Expected update frequency

Annually (or: Never — set to "Never" if you don't plan to refresh it)

## File description (tour_ce_omn12_indices.csv)

Nights spent in short-stay accommodation booked through Airbnb, Booking, Expedia and Tripadvisor, by month, guest residence (Domestic / Foreign / Total) and region (31 countries, 97 NUTS 1, 260 NUTS 2 regions), 2018-2025, with seasonality indices, foreign share, foreign timing effect and geography. 118,794 rows x 33 columns. One row = region x year x month (or annual Total) x guest group. CSV, UTF-8 with BOM; empty field = no value.

## About Dataset (description, Markdown)

## Context

When do guests come to Europe's regions, and how concentrated are their seasons? This dataset takes Eurostat's monthly statistics on **nights spent in short-stay accommodation booked through online platforms** (Airbnb, Booking, Expedia, Tripadvisor; table *tour_ce_omn12*) and adds a set of tested **seasonality indices** for every country, region, year and guest group, plus each region's **foreign share** and **geography**.

It is the data behind the interactive page [Peaks and Plateaus: Europe's Foreign Tourism Rhythms](https://lomska.github.io/AI-Assisted-Dataviz/EUForeignTourismRhythms).

## Content

One file, `tour_ce_omn12_indices.csv`: **118,794 rows × 33 columns**.

* **One row = region × year × month (January … December, or the annual `Total`) × guest group** (`Domestic`, `Foreign`, `Total`).
* **Coverage:** EU-27 plus Iceland, Liechtenstein, Norway and Switzerland — 31 countries, 97 NUTS 1 and 260 NUTS 2 regions (NUTS 2024), **2018–2025**.
* **Column groups:**
  * identifiers and nights (`Region` … `Value`);
  * the 75% layer — which busiest months hold three-quarters of the year (`CumPercentage`, `Prc75Flag`, `ThresholdMonth`, `Season75`);
  * seasonality indices — Gini index, seasonality percentile, one/two seasons, season centre, five season patterns (`VolumeOK` … `SeasonPattern`);
  * foreign share and the foreign timing effect (`PrcForeign` … `ForeignTimingClass`);
  * geography of the residents — urban/rural, coastal, inland mountain, island shares and groups (`PrcCities` … `IslandGroup`).

**Series-level indices are repeated on every row of their series** (e.g. a region-year's Gini is on its 12 monthly rows and its Total row), so filter before aggregating.

## Typical filters

| Purpose | Filter |
|---|---|
| Annual map of regions | `Month == "Total"`, one `Year`, one `NUTSLevel`, one `ResidenceCountry`; join to NUTS boundaries on `NUTSCode` |
| Foreign guests' season length | `ResidenceCountry == "Foreign"`, `NUTSLevel == 2`, `Season75` from any monthly row |
| Monthly profile of a region | `Month != "Total"`, one `NUTSCode`, `Year`, `ResidenceCountry` |
| Country comparison | `NUTSLevel == 0` (country totals can include nights not allocated to any region — don't sum regions) |

## Methods (short)

* **Months to 75% (`Season75`)** — busiest months needed to reach 75% of the year's nights; an even year needs 9. 75% was tested against 50–90%.
* **Gini index** — covariance form of Fernández-Morales, Cisneros-Martínez & McCabe (2016), checked against published example values; 0 = flat year, 0.917 = one month.
* **Minimum volume** — indices only for series with ≥ 10,000 nights a year (`VolumeOK = 1`); below that, year-to-year noise is 3–5 times higher.
* **Season shape and centre** — S_R (Lo Magno, Ferrante & De Cantis 2017) and circular concentration PCD (Zhang & Qian 2003); PCD ÷ S_R < 0.8 = two seasons (rule of this dataset).
* **Season pattern** — five labels from the monthly seasonal factors, cut-offs after Ferrante, Lo Magno & De Cantis (2018); checked for stability and against a Ward clustering.
* **Foreign timing effect** — Gini of all nights minus Gini with foreign nights spread like domestic ones; an accounting comparison, not a causal effect.
* **Geography** — shares of residents from Eurostat's LAU 2024 degree of urbanisation and coastal flag and the NUTS 3 mountain/island typologies.

Full definitions, tests and robustness checks: the three notebooks of the project (transform → indices → findings) and the data dictionary.

## Caveats

* **Platform bookings only** — hotels booked directly and most domestic holiday homes are missing; growth since 2019 partly reflects wider platform use.
* **Foreign guests are one group** — no countries of origin.
* The Dutch and Portuguese NUTS 2024 regions start in **2024**; Greece's Extra-Regio area (Mount Athos) only in 2025.
* **Latvija 2024** (NUTS 1 and 2), Foreign and Total: source error (annual total 268, Oct–Dec near zero), kept as published but without indices — use the Latvia country row.
* Geography shares describe where **residents** live, not where guests stay; no geography for Iceland.
* Month length is not adjusted for (February's fewer days raise the Gini slightly).

## Changes made to the Eurostat data

The original Eurostat table was modified: Eurostat's `:` (not available) inside otherwise published series was set to 0 (51 cells; the months then add up to the published annual totals); 2026 (January–March only) and the superseded NUTS 2021 regions were removed; 6 monthly cells of *Ísland* 2024 that exceeded the country value were replaced by it; all indices, shares and groups were computed for this dataset. **Eurostat bears no responsibility for these modifications or for the indices.**

## Sources

* Eurostat, [tour_ce_omn12](https://ec.europa.eu/eurostat/databrowser/view/tour_ce_omn12/default/table) — nights spent in short-stay accommodation booked via online platforms, monthly; extract of 15 October 2026 (data updated 2 October 2026).
* Eurostat, NUTS 2024 classification and NUTS 3 typologies; LAU 2024 municipalities (population, degree of urbanisation, coastal flag).

## License and attribution

Source data © European Union, 1995–2026, Eurostat, reused under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) in line with Commission Decision 2011/833/EU. This derived dataset is released under CC BY 4.0: please credit **Eurostat (tour_ce_omn12)** and this dataset.

## Methods literature

* Fernández-Morales, Cisneros-Martínez & McCabe (2016). Seasonal concentration of tourism demand: decomposition analysis and marketing implications. *Tourism Management* 56, 172–190.
* Lo Magno, Ferrante & De Cantis (2017). A new index for measuring seasonality: a transportation cost approach. *Mathematical Social Sciences* 88, 55–65.
* Ferrante, Lo Magno & De Cantis (2018). Measuring tourism seasonality across European countries. *Tourism Management* 68, 220–235.
* Zhang & Qian (2003). Annual distribution features of precipitation in China and their interannual variations. *Acta Meteorologica Sinica* 17(2), 146–163.

## Column descriptions (in file order)

| Column | Description |
|---|---|
| Region | Country name (NUTS level 0) or NUTS region name as published by Eurostat. Not unique on its own (e.g. Berlin, Malta appear at several levels): use NUTSCode. |
| NUTSCode | NUTS 2024 code (e.g. DE11); country code for level 0 (EL = Greece). Join key for NUTS boundaries (Eurostat GISCO). |
| NUTSLevel | 0 = country, 1 = NUTS 1, 2 = NUTS 2. |
| ParentRegion | NUTS 2 -> its NUTS 1 region; NUTS 1 -> its country; empty for countries. |
| Country | Country the region belongs to (EU-27 plus Iceland, Liechtenstein, Norway, Switzerland). |
| Year | 2018-2025. |
| Month | January ... December, or Total (annual total). Filter Month == 'Total' for annual rows. |
| ResidenceCountry | Guest group: Domestic, Foreign, or Total (Eurostat's published total over all guests). |
| Value | Nights spent (the measure). Never empty. |
| CumPercentage | Running share (%) of the annual total after ordering the year's months from busiest to quietest (ties in calendar order). Empty on Total rows and for Latvija 2024 Foreign/Total (broken source series). |
| Prc75Flag | 1 for the busiest months that together reach 75% of the year's nights (up to and including the month that crosses 75%, plus any month tied with it), else 0. Empty on Total rows and for Latvija 2024 Foreign/Total. |
| ThresholdMonth | 1 for months tied in nights with the month that crosses 75% (2+ identical months at the break), else 0. Empty as Prc75Flag. |
| Season75 | Months to 75%: number of months with Prc75Flag = 1 in the region-year-guest group (smaller = more seasonal; an evenly spread year needs 9). Same value on all 12 monthly rows; empty on Total rows. |
| VolumeOK | 1 if the series has at least 10,000 nights a year, all 12 months and an annual total equal to the sum of its months; the seasonality indices below are given only where VolumeOK = 1. |
| Gini | Gini index of the 12 monthly values (covariance form, Fernandez-Morales et al. 2016): 0 = all months equal, 0.917 = everything in one month. Ignores the order of months. Differences under 0.02-0.03 are within normal year-to-year movement. Empty where VolumeOK = 0. |
| SeasonalityPercentile | Gini on a 0-100 scale: % of comparable units (same NUTS level, year and guest group; VolumeOK only; Extra-Regio excluded) with a lower Gini. 100 = most seasonal. |
| SeasonShape | One season, Two seasons (separate peaks, e.g. winter + summer) or No clear season (Gini < 0.15). One vs two: circular concentration PCD / S_R (Lo Magno et al. 2017) >= 0.8 = one season. Rule of this dataset; borderline winter-and-summer regions can be classed One season. |
| SeasonCentre | Centre of the year's nights on the calendar circle as a month position (1.0 = January ... 12.0 = December; 12.5 = between December and January). Only for One season years. |
| SeasonCentreMonth | Month nearest to SeasonCentre. Only for One season years; for borderline two-peak years it can fall in a quiet month. |
| SeasonPattern | Five patterns, first matching rule: Mild or no season (Gini < 0.15); Winter-based seasons (two seasons, or centre Nov-Mar); Summer season + winter shoulder (Dec-Feb at least an average month); Strong summer season (Jul-Aug >= 2.5 x average month); Summer season (1.5-2.5); Mild or no season (< 1.5). The small Summer + winter shoulder group is not stable between years. |
| PrcForeign | Foreign / Total nights x 100 for the region, year and month (or annual Total); same value on the Domestic, Foreign and Total rows. Empty for 0/0 months and Latvija 2024 Oct-Dec and annual. |
| ForeignGroup | Mainly domestic (< 40% foreign), Mixed (40 to < 70%), Mainly foreign (>= 70%). |
| ForeignTimingEffect | How much of the region's seasonality lies in foreign guests' timing, in Gini points: Gini of all nights minus the Gini if foreign guests kept their nights but travelled in the same months as domestic guests (= Gini of domestic nights). Positive = foreign timing sharpens the year. Accounting comparison, not a causal effect. One value per region-year; empty unless both guest groups have 12 months and >= 10,000 nights. |
| ForeignTimingClass | Adds (ForeignTimingEffect > +0.03, foreign timing makes the year sharper), No clear effect (-0.03 to +0.03), Reduces (< -0.03, flatter). |
| PrcCities | % of the region's residents living in cities (degree of urbanisation of their municipality, LAU 2024). Same for every year. Towns and suburbs = 100 - PrcCities - PrcRural. |
| PrcRural | % of residents living in rural areas (degree of urbanisation). |
| UrbanGroup | Predominantly urban (PrcRural <= 20%), Intermediate, Predominantly rural (PrcRural >= 50%). |
| PrcCoastal | % of residents in coastal municipalities. |
| CoastalGroup | Non-coastal (0%), Partly coastal (< 50%), Predominantly coastal (>= 50%). |
| PrcInlandMountain | % of residents in NUTS 3 regions that are mountain and not coastal (Eurostat typologies). |
| InlandMountainGroup | Non-, Partly (< 50%), Predominantly (>= 50%) inland mountain. |
| PrcIsland | % of residents in NUTS 3 regions classed as island. |
| IslandGroup | Non-, Partly (< 50%), Predominantly (>= 50%) island. |
