# Indonesia GVA Map

An interactive map of resource-adjusted Gross Value Added (GVA) across Indonesia's 38 provinces, built from official BPS (Statistics Indonesia) national accounts data.

The pipeline estimates provincial GVA at basic prices by industry and tracks its quarterly year-on-year growth from 2020 to 2026. It then overlays the cost of natural resource depletion (coal, oil, gas, copper, timber, and precious and base metals) to produce a resource-adjusted view of regional output, closer to a provincial Net Domestic Regional Product (NDRP) than to headline PDRB/GRDP.

## What it produces

The main output is a self-contained interactive HTML map (`03_outputs/indonesia_gva_depletion_map.html`), built with Folium/Leaflet, showing for each province:

- Provincial GVA (constant 2010 prices) and its YoY growth
- Estimated resource depletion cost and the resulting "haircut" to output
- Resource-adjusted regional output
- A click-through panel with a GVA sparkline, sectoral bar chart, and a waterfall chart of sector contributions to growth

## Methodology

1. **National GVA/tax ratios from the 2020 Input–Output Table.** BPS's 2020 IOT (17 industries, domestic transactions at basic prices) is used to derive, per sector, the ratio of GVA at basic prices to GDP at market prices: `gva_multiplier = gva_basic / gdp_market`.
2. **Provincial PDRB cleaning.** Quarterly provincial PDRB ("Produk Domestik Regional Bruto Menurut Lapangan Usaha") releases from 2020–2026 are parsed and consolidated for both current prices (ADHB) and constant 2010 prices (ADHK).
3. **Provincial GVA estimation.** The national sector multipliers are applied to provincial PDRB, sector by sector, for every province and period: `GVA_provincial[sector] = PDRB_provincial[sector] × gva_multiplier[sector]`. ADHK is used for all growth calculations (it strips out price effects); ADHB is retained for structural composition (GVA as a share of GDRP).
4. **Resource depletion overlay.** Depletion costs by commodity (SISNERLING environmental-economic accounts, 2020–2024) are allocated to provinces using estimated commodity production shares, then subtracted from provincial GVA to give a resource-adjusted figure.
5. **Mapping.** Provincial GVA, growth, and depletion figures are joined to provincial boundaries and rendered as an interactive choropleth.

**Key assumption:** sector-level GVA/tax multipliers from the single 2020 IOT are held constant across all provinces and all years (a "frozen coefficient" approach). Provincial tax structures are assumed to match the national average by sector.

## Data sources

- **BPS (Badan Pusat Statistik)**
  - *Tabel Input-Output Indonesia 2020*: Transaksi Domestik Atas Dasar Harga Dasar (17 industries)
  - *[Seri 2010] PDRB Triwulanan Atas Dasar Harga Berlaku & Atas Dasar Harga Konstan Menurut Lapangan Usaha di Provinsi Seluruh Indonesia*: quarterly provincial GRDP by industry, current and constant prices, 2020–2026 releases
- **SISNERLING**: environmental-economic accounts, natural resource depletion series (coal, oil, gas, copper, timber, gold, silver, tin, nickel, bauxite), 2020–2024
- **Provincial boundaries**: GeoJSON sourced from [superpikar/indonesia-geojson](https://github.com/superpikar/indonesia-geojson) (GADM level 1 is used as a higher-quality alternative)

## Repository structure

```
00_base_data/           Raw BPS CSVs: the 2020 IOT and yearly PDRB Triwulanan releases (2020–2026)
01_scripts/              Notebooks, run in numeric order, plus the provincial GeoJSON
  01_01_indonesia_gva_etl.ipynb            Derive sectoral GVA/tax ratios from the 2020 IOT
  02_01_regional_gdpr_cleaning.ipynb       Clean & consolidate provincial PDRB (ADHB + ADHK)
  03_01_provincial_gva_calculations.ipynb  Apply IOT multipliers to estimate provincial GVA + YoY growth
  04_01_visualisations.ipynb               Exploratory charts (line, bar, waterfall, structural share)
  05_01_indonesia_gva_map.ipynb            Build the resource-adjusted interactive HTML map
02_intermediate_data/    Cleaned and derived CSVs produced between pipeline stages
03_outputs/               Final interactive HTML map
```

## Getting started

**Requirements:** Python 3, with `pandas`, `numpy`, `matplotlib`, `seaborn`, `ipywidgets`, `folium`, and `branca`.

```bash
pip install pandas numpy matplotlib seaborn ipywidgets folium branca
```

1. Clone the repo and open the notebooks in `01_scripts/` in a Jupyter environment.
2. Each notebook currently points to a local Windows path (e.g. `C:\Users\Admin\...`). Update the `BASE`/`DATA_DIR`/`INTER` path variables near the top of each notebook to point to your local clone.
3. Run the notebooks in order (`01_01` → `05_01`). Each stage reads the previous stage's output from `02_intermediate_data/`.
4. Open `03_outputs/indonesia_gva_depletion_map.html` in a browser to view the map.

## Limitations

- **Frozen coefficients:** GVA/tax multipliers come from a single national 2020 IOT and are applied uniformly to every province and year; they don't capture provincial variation in tax or subsidy structure.
- **Provincial coverage:** the four newest Papua provinces (Papua Barat Daya, Papua Selatan, Papua Tengah, Papua Pegunungan) have no BPS sectoral PDRB before 2023, so the full time series covers 34 provinces, expanding to 38 from 2023 onward.
- **Depletion allocation:** provincial shares of national resource depletion are estimated from mining-sector PDRB and production geography, not official province-level SISNERLING figures.
- **Release lag:** ADHB (current price) data runs through 2026Q1; ADHK (constant price) data runs only through 2025Q4, as the corresponding release wasn't yet available.

## License

Apache License 2.0. See [LICENSE](LICENSE).
