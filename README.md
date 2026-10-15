# Three rounds of community sport

One page about Queensland Active Clubs approval rounds in 2022, 2023 and 2024. The implementation uses the same Vega, Vega-Lite and Vega-Embed versions as the Week 9 lecture sample. No additional browser libraries, build system or Python script are required to run this version.

## Open the page

Open `index.html` using VS Code Live Server. Loading through a web server is important because the chart files fetch CSV and GeoJSON. The three Vega libraries are loaded from jsDelivr and need internet access.

## Small repository structure

```text
index.html                         Story and six simple vegaEmbed calls
css/styles.css                     Page layout and chart control styling
charts/funding-map.vg.json          Count records per council; look up map coordinates
charts/football-growth-lollipop.vg.json Compare percentage changes since 2022 in two gender panels
charts/asf-sport-state-map.vg.json    Filter published sport percentages; look up state geometry
charts/annual-funding.vg.json       Sum approved funding by year; display three bars
charts/sport-bump.vg.json           Count rows by sport/year; rank and choose top 5 or 10
charts/sport-purpose-heatmap.vg.json Count sport/purpose pairs; use square-root colour contrast
data/raw/active-clubs-round-2.xlsx   Original 2022 source workbook
data/raw/active-clubs-round-3.xlsx   Original 2023 source workbook
data/raw/active-clubs-round-4.xlsx   Original 2024 source workbook
data/raw/active-clubs-records.csv    Combined original rows in a browser-readable format
data/raw/football-age-gender-2022-2025.csv Published age/gender counts extracted from four PDFs
data/raw/asf-sports-by-state-2023.csv Published percentages transcribed from ASF page 28
maps/australia-states-2021.geojson   ABS state boundaries for the national survey map
maps/qld-lga-2024.geojson            ABS geography and representative council coordinates
```

The old `data/processed/` files are not referenced. They may be removed. The previous preparation script and page-interaction script are not needed by this version.

## What “raw data” means here

Vega-Lite cannot directly load Excel workbooks. The CSV is a format conversion and concatenation of their original funding rows, not an aggregated or cleaned summary. It keeps all twelve source columns and all 5,563 records: 1,690 in 2022, 2,116 in 2023 and 1,757 in 2024. Repeated-looking source rows are kept. Empty spreadsheet rows are not records and are omitted.

Two representation adjustments are documented: the first column uses `Program Round` consistently (Round 3 originally says `Program Name`), and Excel dates are written as `dd/mm/yyyy`. Category wording and funding amounts remain as supplied. The original workbooks stay alongside the CSV for checking.

To reproduce the CSV without programming: save each workbook's data sheet as CSV in Excel, use the same first-column header, put the rows underneath one shared header, and save the combined file as UTF-8 CSV. Preserve dates as day/month/year. For later updates, replace the CSV with the new original rows rather than manually changing chart totals.

## Explain the chart calculations in an interview

All counts, ranks, selected categories and totals are calculated in the chart JSON `transform` arrays:

- `calculate` extracts the approval year and gives original fields short names within the chart.
- `filter` selects a year or the leading categories.
- `aggregate` counts funding records and sums the funding field.
- `joinaggregate` attaches each sport's or purpose's total to its rows.
- `window` orders categories and assigns ranks. The bump chart ranks all sports before filtering the chosen top group.
- `lookup` matches council keys with the geographical reference file.
- `impute` completes empty sport/purpose combinations with zero counts for the heatmap; it does not add funding records to the source.

Selectors and the bump-chart legend highlight are defined through Vega-Lite `params`. The HTML's JavaScript only embeds the nine charts, like the lecture example. No custom event-handler code is needed.

## What the data can support

Each source row is a funding record. A sport is a funded activity category. These workbooks do not count events held, participants or club members. The funding bars compare totals for three approval rounds, in Australian dollars excluding GST. All Round 2 records have $2,000 funding and the later rounds have $2,500 per row; this increase in grant size also affects annual funding totals.

The funding map uses equal-sized dots and the same 2024 boundaries for all years. Colour shows record count on a fixed square-root scale from 0 to 350. This expands colour differences among smaller counts while preserving a common scale across years. Dot coordinates represent councils rather than club addresses. One 2022 record has a blank council field and remains in annual totals but cannot be mapped. Council casing/suffix differences are matched in the map specification, not rewritten in the CSV.

The bump chart selects top activities using total records across all three rounds. Annual ranks are among all categories. Ties are ordered alphabetically. Rank and absolute funding-record counts answer different questions; the tooltip gives both.

The heatmap selects the top five sports and original purposes separately in each year. Purpose names differ between releases, so no harmonisation is implied. Cell labels and tooltips show original counts. A square-root colour scale uses zero and the largest displayed count in each selected year, expanding colour differences among smaller values. Colours are relative to the selected year; compare labels for absolute differences across years. A grouped maximum supplies the threshold for readable text colour. Axis labels omit long source prefixes; the tooltip retains the full wording.

## Why these idioms

The coloured council-dot map answers where support is located and how record counts differ; the funding bar chart compares total approved dollars by year; the bump chart follows changing activity ranks; the heatmap compares activity and purpose. Sankey and sunburst were considered. This dataset does not record flows between activities, and an additional hierarchical layout would make the implementation harder to explain without adding a clearer comparison. No extra Sankey library is included.

## Sources

- Queensland Government, [Active Clubs](https://www.data.qld.gov.au/dataset/active-clubs), supplied Round 2, 3 and 4 workbooks, CC BY 4.0.
- ABS, [ASGS 2024 LGA boundaries](https://geo.abs.gov.au/arcgis/rest/services/ASGS2024/LGA/MapServer), Queensland geography simplified for web display, CC BY 4.0. Representative coordinates in the map file are calculated from council geometry, not from club locations.
- [Vega-Lite data formats](https://vega.github.io/vega-lite/docs/data.html), [ranking transforms](https://vega.github.io/vega-lite/docs/window.html).

## ASF state choropleth

This is a separate national survey section, not a join with Queensland funding records. `data/raw/asf-sports-by-state-2023.csv` transcribes all 36 sport rows and eight location columns from page 28 of ASF *Clubs Under Pressure*, May 2023, including the state response bases. It is a published summary, not respondent-level raw data. Each percentage is preserved exactly; no totals are forced to 100% and no absolute club counts are estimated. `source_row` preserves the two separate source rows labelled Skateboarding. Higher/lower significance markers are stored separately as `significance`; they indicate subgroup comparisons, not time trends, and are not used for colouring.

The selector filters one source sport row. `lookup` attaches its state's geometry. `calculate` divides the published percentage by 100 so Vega-Lite can format it as a percentage. The choropleth uses a linear fixed 0–50% scale; the largest published value is 44%. Tooltips include state response bases. Map label coordinates are cartographic positions, not club addresses. The ACT label is offset with a connector for legibility.

The state percentages exceed 100% in some columns; do not interpret categories as mutually exclusive. Page 28 does not explain the multiple-category mechanism, so the webpage avoids asserting a reason. Rounded 0% need not mean no respondents. ACT and NT have much smaller bases. This survey table cannot establish membership growth or effects of funding.

Sources: [ASF report page](https://asf.org.au/resources/clubs-under-pressure); [ABS ASGS 2021 state boundaries](https://geo.abs.gov.au/arcgis/rest/services/ASGS2021/STE/MapServer), layer 1, generalised, CC BY 4.0. The geometry file records its download query. The source PDF remains in your Downloads folder; no PDF-reading or CSV-processing library is needed by the website.

## Football growth lollipop chart

Section 06 now loads `charts/football-growth-lollipop.vg.json`, a Vega-Lite layered lollipop chart. The same simple HTML `vegaEmbed` call is used; no additional library is needed. It reuses `football-sunburst-counts-2022-2025.csv`, including MiniRoos, Youth and Senior. The previous Youth/Senior-only CSV is preserved as a source extract.

First `calculate` records the 2022 count in `BaselineInput` and null for other years. `joinaggregate` attaches that count to all rows sharing Region, AgeGroup and Gender. The maximum returns the one non-null baseline; it is not the maximum participant count across years. Another calculation gives `Growth = Participants / Baseline - 1`. Only after this calculation does `filter` select the requested age group and comparison year. Nonpositive baselines would produce an unavailable change rather than an invented percentage; all supplied baselines are positive.

`facet` separates Male and Female into two panels. A `rule` runs between 0 and Growth, a filled point marks the endpoint, and text shows the exact percentage change to one decimal place. Teal indicates growth/no change; orange indicates decline. Each panel uses the same region order and horizontal scale. The zero line separates decreases from increases. The axis is fixed across comparison years within an age category: -50% to +180% for MiniRoos, -50% to +100% for Youth, and -50% to +70% for Senior. Both gender panels share that scale. These bounds contain all supplied changes and leave room for labels. Fixed panel widths allow horizontal scrolling on small screens.

Tooltips retain current counts, 2022 counts and the selected-year source page. Percentages describe relative changes in reported participant counts, not population rates or percentage points. Small starting counts can produce large relative increases. NSW and Northern NSW remain separate reporting regions; genders exclude unspecified participants. The chart compares one selected year against 2022, rather than displaying all intermediate years simultaneously.

## Publish with GitHub Pages

Commit the page, stylesheet, nine chart specifications, all five CSVs, the original Ballarat Sports Grounds JSON and the map files used by the other maps. Enable GitHub Pages for the repository's main branch and root folder. Original workbooks may be committed too so the source downloads on the page work.

## Ballarat site-by-type matrix

Section 07 uses `charts/ballarat-site-type-matrix.vg.json`, a Vega-Lite matrix with rect cells and count labels. It loads the user's original `data/raw/Ballarat Sports Grounds.json` directly using `format.property: "features"`. No coordinate conversion, prepared count table, new script or browser library is required. The geographical export is retained alongside the original source, but this chart needs only the Site and Feat_Type properties.

`calculate` reads the Site field and removes the introductory SG- from type labels. `aggregate` counts features for each Site/GroundType combination. `joinaggregate` sums those counts within each site. `window` with dense_rank ranks sites by total count descending and site name ascending; all type rows at a site receive the same rank. The selector filters the top 10, top 20 or all 36 sites. `impute` creates missing combinations with zero chart counts using the ten known source categories; it does not add source features. Rows follow the same total-count ranking and columns keep the same type order for every selection.

The original data has 146 features, 36 site names and 10 types. The largest site/type cell contains 19 features, so the fixed square-root colour scale runs from 0 to 20. Rect colour and text show cell counts; blank pale cells are zero source matches. Tooltips show cell count, site total and rank. Equal site totals are ordered alphabetically. Combined categories such as Cricket/Football remain intact.

A site can have several features of the same type. Feature totals are not club, participant or event counts. Zero matches do not prove real-world absence or unavailability. The source remains a snapshot, not a time series. Source: [City of Ballarat Sports Grounds](https://ballaratopendata.org.au/dataset/ballarat-sports-grounds), CC BY 3.0 Australia. The geographic dot-map specification is retained as an unused alternative; the webpage embeds the matrix.

## Football three-ring sunburst

`charts/football-sunburst.vg.json` uses **Vega**, not Vega-Lite. Vega's built-in `stratify` and `partition` transforms create the hierarchy and sunburst layout. The existing Vega and Vega-Embed libraries already support this; no extra library, plugin or custom HTML event handler is added. The HTML still uses one simple `vegaEmbed` call. Check whether the assignment allows Vega specifications if it specifically requires every chart to use Vega-Lite.

The CSV preserves 216 published counts: 4 years × 9 reporting regions × 3 age categories × 2 genders. MiniRoos counts are transcribed directly from all 36 regional summary panels in the source PDFs, alongside the previously extracted Youth/Senior counts. No values are estimated from rounded percentages. `SourceFile`, `SourcePage` and `AgeGroupTotal` provide source references; the original PDFs remain in Downloads. No extraction script is required by the webpage.

The region selector filters the original records. Three `aggregate` transforms sum them by age, age/gender and age/gender/year. Nodes receive IDs and parent IDs. `stratify` links those nodes; `partition` assigns angular spans in proportion to the leaf counts. Internal nodes have `LeafValue = 0` so parent totals are not counted again when calculating layout. Their separately computed `Participants` values supply accurate tooltips. The root is hidden, leaving exactly three visible rings.

Inner ring: MiniRoos / Youth / Senior. Middle ring: Male / Female. Outer ring: report year. Age colours remain fixed; female branches are lighter. Small segments have hover details even when their labels are hidden. The chart has a fixed width and can scroll on narrow screens.

The centre and inner two rings sum counts across 2022–2025, which is not a count of unique people or general population. A returning participant can appear in several years. Gender-unspecified counts are excluded rather than inferred. NSW and Northern NSW are separate reporting regions. Section 06 complements the sunburst with a lollipop comparison of regional percentage growth since 2022.

## AusPlay participant-count spike map

`charts/ausplay-participant-spike-map.vg.json` uses Vega-Lite and the existing simple `vegaEmbed` call. No new library is included. The CSV preserves the eight state Total estimates and rates from the supplied screenshot, with All period, Adult population, All gender and All age. No reporting dates are invented. The national Total is excluded from the geographical symbols.

`lookup` supplies representative cartographic spike bases. `calculate` sets `SpikeHeight = estimate / 2000000 * 160`, so one million participants is 80 pixels tall. A second calculation builds a triangular SVG shape string from that height, with a fixed 10-pixel base. Vega's existing symbol renderer accepts this path; mark size 4 gives the path its original dimensions. The `shape` scale is disabled so Vega-Lite uses the path directly. Zero participants would give zero spike height. This encodes quantity with height rather than symbol area. Labels sit below the spike bases; tooltips give counts and participation rates. ACT is offset with a connector. The map uses a fixed width so its pixel height scale stays stable.

Spikes show estimated adult participants, not general population or club memberships. Their bases represent states, not addresses. State population size affects absolute counts. The screenshot's Period is All; choose a named period and update the data before labelling it with dates.

## Funding map dots

The first map now uses one equal-sized dot per council with mapped records, not one dot per funding record. `mark.size = 36` fixes the dot area in screen pixels. The previous quantitative `size` encoding is replaced with `color` for `Records`; a square-root colour scale has the same 0–350 domain in all three years. No record values or representative positions are changed. Exact counts and funding totals remain in hover details. Smaller dots reduce overlap but do not guarantee that every nearby council position is separated. A missing dot does not establish no sporting activity or no clubs.

The funding dots use a yellow/orange/red palette with a fixed maximum of 350 across all years. The largest mapped source count is 304. The square-root transformation and wider palette make low and medium counts more distinct without changing their values.
