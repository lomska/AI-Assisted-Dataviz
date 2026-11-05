# Peaks and Plateaus: Europe's Foreign Tourism Rhythms

[Explore the Page](https://lomska.github.io/AI-Assisted-Dataviz/EUForeignTourismRhythms)

[![Peaks and Plateaus: the map, the country-by-month grid and the season patterns](preview.png)](https://lomska.github.io/AI-Assisted-Dataviz/EUForeignTourismRhythms)

When do foreign guests come to Europe's regions?
How concentrated are their seasons?
Do they make a region's year more or less seasonal than domestic guests do? 

An interactive page exploring these questions for 31 countries and their NUTS 2 regions, 2025. All figures describe stays booked through online platforms (Airbnb, Booking, Expedia, Tripadvisor), not all tourism.

The folder contains three notebooks: 

| Path | Content |
|---|---|
| `Stage 1 Transform.ipynb` | Raw Eurostat files → clean, tested nights |
| `Stage 2 Indices.ipynb` | Seasonality and foreign-share indices, with robustness checks |
| `Stage 3 Analysis.ipynb` | Findings & conclusions (`.html` export alongside) |

## Sources

- Eurostat, [tour_ce_omn12](https://ec.europa.eu/eurostat/databrowser/view/tour_ce_omn12/default/table): nights spent in short-stay accommodation booked via online platforms, monthly, extract of 15 October 2026
- Eurostat NUTS 2024 classification and LAU 2024 municipalities (geography types)
- Map outlines: Eurostat GISCO, NUTS 2024, 1:20M, via [Nuts2json](https://github.com/eurostat/Nuts2json)
