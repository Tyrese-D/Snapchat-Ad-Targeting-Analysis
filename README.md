# The Data Behind the Lens: Analyzing Snapchat's Ad Targeting

## Project Overview
This project analyzes Snapchat's political and advocacy ad targeting database
to uncover patterns in how advertisers use demographic targeting criteria,
how ad spend correlates with major political events, and what ethical concerns
emerge from targeting practices directed at young and minor audiences. The
analysis spans 2,927 unique ad records across 29 countries, $3,893,165 in
total ad spend, and 1,725,925,240 total impressions collected across 2018
and 2019.

LIVE DASHBOARD:https://public.tableau.com/app/profile/tyrese.dieudonne/viz/SnapchatAdTargetingAnalysis/SnapshotAdsDashboard
 
  `.value_counts()`
- **Data Sources:**
  - `Snapchat_Ads.csv` — 2,927 ad records across 52 columns, primary
    analysis dataset
  - `Snapchat_Ads_2_Multiple_Connections.csv` — 35,124 pivoted records
    used for targeting dimension analysis and seasonal trend tracking
  - `Sheet1_Overtime_data.csv` — 5,854 records enriched with election
    type classifications, notable dates, and start/end date pivots used
    for the year-over-year trend line

## The Data Analysis Process
I followed a 3-stage ETL framework across three interconnected datasets to
move from raw ad records to a finished analytical dashboard:

1. **Extract:** Loaded all three CSV files into pandas DataFrames as staging
   environments, confirmed shapes of 2,927 × 52, 35,124 × 28, and
   5,854 × 38, respectively, and immediately flagged the country name casing
   inconsistencies, high null rates across optional targeting fields,
   age brackets that included minors as young as 14, 60 ads with zero
   spend, and a Location targeting flag with three distinct values instead
   of the expected binary 0/1
2. **Transform:** Applied all cleaning operations across seven categories —
   null handling and structural interpretation, country name standardization,
   targeting flag validation, age bracket minor flagging, zero-spend
   investigation, date type conversion and election period enrichment,
   and dataset joining for the pivot-based seasonal analysis
3. **Dashboard Construction:** Built the full Tableau dashboard across five
   visualizations — a KPI summary, country bar chart, seasonal ad volume
   trend line, audience segmentation donut charts, cross-market spend
   comparison, and a targeting growth and ethical concern slope chart

## Data Cleaning Steps

### Null & Missing Values
- **Null Profile:** High null rates existed across many columns — but most
  were structurally expected rather than errors:
  - `Advanced Demographics` — **96.9% null** (2,836 of 2,927 records) —
    this field is only populated when an advertiser actively uses advanced
    demographic targeting such as marital status or parental status; null
    means the targeting method was not used, not that data is missing
  - `Electoral District` — **97.8% null** (2,862 records) — only populated
    for ads explicitly tied to a specific electoral race; most ads are
    national or regional, not district-level
  - `CandidateBallotInformation` — **97.1% null** (2,842 records) — only
    populated for candidate-specific political ads; advocacy and issue ads
    Leave this blank by design
  - `Gender` (targeting field) — **89.4% null** (2,617 records) — gender
    Targeting was used in only 310 ads out of 2,927 total
  - `Interests` (targeting field) — **76.8% null** (2,249 records) —
    Interest-based targeting is used in fewer than a quarter of all ads
  - `Language` — **72.5% null** (2,123 records) — language targeting used
    in roughly 800 ads, primarily in multilingual countries
  - `EndDate` — **20.9% null** (611 records) — ads without an end date
    were treated as open-ended campaigns with no scheduled termination
  - `Metro Area` — **94.2% null** (2,758 records) — metro-level targeting
    used in only a small fraction of campaigns
- **Handling Decision:** Did not impute or drop any of these null fields —
  each null carries a specific analytical meaning (targeting method not
  used) and replacing them with placeholder values would have falsely
  inflated targeting usage rates across every dimension in the dashboard
- **Recoded for Analysis:** Null values in targeting flag columns (`Age`,
  `Gender`, `Language`, `Interests`, `Segments`, `Advanced demographics`,
  `OS type`, `Location`) were all stored as binary 0/1 indicators —
  confirmed that all returned only `[0, 1]` except for `Location` which
  returned `[0, 1, 2]`, indicating a three-tier location targeting
  intensity that required separate documentation

### Duplicate Records
- **Duplicate Check:** Ran `df.duplicated().sum()` across all three datasets
- All three files returned **zero exact duplicate rows** — every ad ID
  appeared exactly once in the primary dataset
- Verified using `Ad ID` as a unique key — all 2,927 Ad IDs in
  `Snapchat_Ads.csv` were distinct hashed identifiers
- The same Ad IDs appearing across multiple files (the pivoted and overtime
  datasets) were by design — each file represents a different analytical
  view of the same underlying ad records, not duplicate data

### Country Name Standardization
- **Discovery:** Ran `.value_counts()` on the `Country` column and found
  inconsistent casing across all 29 countries — values like `united states`,
  `United states`, `Netherlands`, `netherlands`, and `United kingdom` all
  appeared for the same countries
- **Impact:** Inconsistent casing caused the same country to appear as
  multiple distinct values in any group-by operation — `united states` and
  `United states` would be counted as two separate countries, splitting the
  1,667 US records and misrepresenting the country breakdown bar chart on
  the dashboard
- **Resolution:** Standardized all country names to Title Case using
  `.str.title()` and `.str.strip()` — resolving all casing variants into
  single consistent labels for each of the 29 countries
- **Dashboard Impact:** The Countries KPI showing **29 unique countries**
  and the country bar chart showing United States leading at 1,667 records
  both depend on this standardization — without it those counts would have
  been artificially inflated by casing duplicates

### Targeting Flag Validation
- **Binary Flag Audit:** Validated all eight targeting dimension columns —
  Age, Interests, Gender, Language, OS Type, Advanced Demographics,
  Segments, and Location — by checking unique values in each
- Seven of the eight returned exactly `[0, 1]` as expected — a 0 meaning
  the targeting dimension was not used, a 1 meaning it was
- **Location Exception:** The `Location` column returned three unique values
  — `[0, 1, 2]` — indicating a three-tier targeting intensity rather than
  a simple binary used/not-used flag
  - `0` — no location targeting applied
  - `1` — broad location targeting (country or region level)
  - `2` — granular location targeting (metro area, zip code, or
    electoral district level)
- Documented this distinction explicitly so the audience segmentation
  donut charts on the dashboard correctly reflected location as a
  multi-intensity dimension rather than a simple binary

### Age Bracket Minor Flagging
- **Discovery:** Ran a scan of all unique `AgeBracket` values and found
  **37 distinct age brackets that included individuals under 18**,
  including ranges starting as low as 14 years old:
  - `14-16`, `14-17`, `14-18`, `14-19`, `14-20`, `14-21`, `14-25`
  - `15-17`, `15-18`, `15-19`, `15-20`, `15-22`, `15-23`, `15-24`
  - `15-25`, `15-30`, `15-34`, `15+`, `16+`, `17+`, `17-24`, `17-29`
  - and more
- This was not treated as a data error — these are the actual age brackets
  recorded in Snapchat's political ad transparency database, meaning
  advertisers actively chose to target these age ranges
- Created a boolean flag column `targets_minors = True` for all records
  where the age bracket lower bound was below 18
- This flag powered the ethical concern analysis visible in the dashboard
  finding that **75% of targeted advocacy spending** reached younger
  demographics with some campaigns reaching individuals as young as
  **14 years old**

### Zero-Spend and Extreme Spend Investigation
- **Zero Spend:** Found **60 ads with $0 in spend** — investigated and
  determined these were ads that were created and approved in the system
  but never actually ran, or test ads submitted without budget allocation
  — retained them in the dataset but excluded from all spend-based
  calculations and the Total Spend KPI
- **Extreme Spend:** Found **66 ads with spend exceeding $10,000**, with
  the single highest spend at **$133,075** — attributed to a US-based
  advertiser (Everytown for Gun Safety AF) and verified against the
  cross-market comparison chart on the dashboard
- The wide spend distribution — mean of $1,330 but a median of only $169
  — confirmed a heavily right-skewed distribution where a small number
  of high-budget advertisers account for the majority of total spend
- General Mills was the top overall spender at **$509,621**, followed by
  Cossette Media Inc at **$400,304** and Assembly at **$283,563**

### Date Type Conversion & Election Period Enrichment
- **Type Conversion:** `StartDate` and `EndDate` were loaded as plain
  strings — converted both to proper datetime objects using
  `pd.to_datetime()` so date arithmetic and time-series aggregations
  could be performed correctly
- **Election Period Classification:** Using the `Sheet1_Overtime_data.csv`
  enrichment file, classified each ad by election context:
  - **Election type 2018** — all 3,334 classified records were `Midterm`
  - **Election type 2019** — classified as Off-year (3,334), Local
    elections (688), Federal (586), European Parliament (224), and
    General elections (194)
  - Notable dates like `29th and 7th October` were tagged to specific
    ad surges visible in the Seasonal Influence trend line
- **Overtime Dataset Pivot:** The `Sheet1_Overtime_data.csv` file used a
  pivot structure where each ad appeared twice — once with `ENDDATE` and
  once with `StartDate` as the `Pivot Field Names` value — creating the
  time series used to track running sum of SD targeting across the
  Aug 2018 to Jun 2020 date range visible on the dashboard

### Targeting Criteria Count Validation
- **Discovery:** The `No. of target critereas used` column recorded how
  many targeting dimensions each ad applied simultaneously — validated
  the full distribution:
  - `0 criteria` — 11 ads (completely untargeted, broadcast ads)
  - `1 criteria` — 291 ads
  - `2 criteria` — 974 ads (largest group)
  - `3 criteria` — 926 ads
  - `4 criteria` — 610 ads
  - `5 criteria` — 92 ads
  - `6 criteria` — 22 ads
  - `7 criteria` — 1 ad (maximum observed targeting complexity)
- Confirmed the dashboard finding that **90% of political ads utilized
  at least two specific targeting criteria** — only 302 out of 2,927 ads
  used zero or one targeting dimension
- No invalid values (negatives, nulls, or values above 7) were found —
  the column was clean

## Dashboard Process

### Step 1 — Connecting the Data Sources
- Connected all three CSV files into Tableau as separate data sources
  and established relationships between them using `Ad ID` as the
  common key across all three files
- Set the join between the primary and pivoted sources as a LEFT JOIN
  to preserve all 2,927 primary ad records regardless of whether they
  had a matching row in the pivot table
- The overtime enrichment file was joined on `Ad ID` to bring election
  type classifications and notable date annotations into the time series
  view without duplicating the primary ad records

### Step 2 — Calculated Fields and Data Preparation in Tableau
- **Country Standardization:** Created a calculated field applying
  title case normalization to the `Country` dimension to prevent
  casing variants from appearing as separate members in filters and charts
- **Minor Targeting Flag:** Built a calculated boolean field that
  evaluated the lower bound of each `AgeBracket` value and returned
  True for any bracket starting below age 18 — used to color-code
  records and filter the ethical concern section of the dashboard
- **Targeting Intensity Bucket:** Created a calculated field grouping
  `No. of target critereas used` into Low (0-1), Medium (2-3), and
  High (4+) buckets to power the intensity breakdown in the key
  findings text block
- **Zero Spend Filter:** Applied a data source filter excluding all
  records where `Spend = 0` from every spend-based measure so the
  Total Spend KPI and all averages reflected only ads that actually ran
- **LOD Calculation for KPI:** Applied a Fixed Level of Detail
  expression to the total spend measure so the KPI card value remained
  anchored to the full dataset regardless of what dimension filters
  were applied to individual charts below it

### Step 3 — KPI Summary Row
- Built three individual KPI tiles using distinct country count, sum
  of spend, and sum of impressions as the underlying measures
- Formatted spend as currency and impressions with comma separators
  for readability at a glance
- Placed all three tiles in a horizontal container at the top of the
  dashboard with consistent font sizing, no borders, and no chart
  chrome so they read as clean headline numbers

### Step 4 — Country Bar Chart
- Built a horizontal bar chart ranked by number of ad records per
  country in descending order so the United States at 1,667 records
  appeared at the top
- Applied direct count labels to each bar and added a click-based
  filter action so selecting any country bar immediately filtered all
  other charts on the dashboard to that country's records

### Step 5 — Seasonal Influence: Ad Volume Trends
- Used the overtime enrichment dataset as the source for this chart
  to access the pivoted date structure where each ad contributed two
  date points — a start and an end — creating a continuous time series
- Applied a running sum table calculation on top of the weekly spend
  aggregation to show cumulative targeting volume building over time
  rather than point-in-time spend which would have appeared too noisy
- Filtered the line breakdown to the five most active countries to
  prevent overcrowding and added reference band annotations marking
  the pre-election spike periods for the 2018 US Midterms and the
  2019 European Parliament elections

### Step 6 — Audience Segmentation Donut Charts
- Built six individual donut charts — one per targeting dimension —
  using a dual-axis technique where the outer ring showed the percentage
  of ads using that dimension and an inner white filled circle masked
  the center to create the donut shape
- Calculated each percentage using the targeting flag column divided
  by the total record count, formatted to two decimal places
- Arranged all six in a 2×3 grid inside a horizontal container with
  consistent sizing, spacing, and color palette

### Step 7 — Cross-Market Comparison
- Built two side-by-side horizontal bar charts filtered to the United
  States and United Kingdom respectively, each ranked by total spend
  per organization in descending order
- Added country dropdown parameter controls on each side so viewers
  could swap the comparison markets without rebuilding the view
- Placed a `Vs` label between the two charts using a text object inside
  the layout container to make the side-by-side structure explicit

### Step 8 — Trajectory of Targeting Slope Chart
- Built a slope chart with year (2018 and 2019) as the two axis points
  and total spend per organization as the measure, with one line per
  advertiser connecting their spend across the two years
- Colored lines using the minor targeting boolean flag so organizations
  that targeted users under 18 were visually distinct from those that
  did not — making the ethical concern immediately visible without
  requiring the viewer to read supporting text first
- Added the key findings annotation block to the right of the chart
  summarizing the 75% finding and the 14-year-old minimum age discovery

### Step 9 — Layout, Filters, and Publishing
- Assembled all charts in a fixed-width vertical layout container with
  consistent padding, a white background, and the project title and
  introductory paragraph as a formatted text header
- Applied a global year filter parameter at the top of the dashboard
  so all charts responded simultaneously to the selected year
- Enabled cross-chart filter actions on the country bar chart and the
  cross-market comparison panels
- Published the completed dashboard to Tableau Public

## SQL Process

### Schema Design
- Designed a three-table relational schema reflecting the three source
  files — a primary ads table, a targeting pivot table, and an election
  enrichment table
- The primary ads table used `ad_id` as the PRIMARY KEY since every
  ad has a unique hashed identifier — this enforced row-level uniqueness
  at the database level and made all future joins reliable
- The pivot and enrichment tables used `ad_id` as a FOREIGN KEY
  referencing the primary table, establishing the correct one-to-many
  relationship between a single ad and its multiple pivot rows or
  election annotations
- Enforced data type precision at the schema level — spend stored as
  DECIMAL rather than FLOAT to prevent rounding errors in financial
  aggregations, dates stored as TIMESTAMP rather than VARCHAR so all
  date arithmetic works natively, and boolean flags stored as BOOLEAN
  rather than integers to make filtering intent explicit

### Constraints and Data Quality Enforcement
- Added CHECK constraints directly into the schema to prevent invalid
  data from entering the database on any future load — spend constrained
  to be greater than or equal to zero, location targeting tier
  constrained to only accept the three valid values (0, 1, 2), and
  targeting criteria count constrained between 0 and 10
- These constraints act as a permanent guardrail so data quality issues
  from upstream source systems are caught at ingestion rather than
  discovered months later when a dashboard metric produces an unexpected
  result

### KPI and Aggregation Queries
- Wrote aggregation queries to produce the three dashboard KPI values —
  distinct country count, total verified spend excluding zero-spend
  records, and total impressions — using conditional filtering to ensure
  zero-spend ads were excluded from financial metrics while remaining
  in the dataset for ad volume counts
- Used window functions alongside standard aggregations to calculate
  each country's percentage share of total spend in a single query pass
  rather than requiring a subquery or a second table scan

### Targeting Analysis Queries
- Wrote queries to calculate the percentage of ads using each of the
  eight targeting dimensions by summing the binary flag columns and
  dividing by the total record count — producing the exact percentages
  shown in the audience segmentation donut charts
- Queried the full targeting criteria count distribution to confirm the
  finding that 90% of ads used two or more criteria simultaneously

### Minor Targeting Queries
- Queried all records where the `targets_minors` flag was True to
  produce a complete list of organizations that ran age brackets
  including users under 18, ranked by total spend
- Calculated the share of total impressions and spend directed at
  minor-inclusive audiences versus adult-only audiences to quantify
  the scale of the ethical concern rather than leaving it as a
  qualitative observation

### Seasonal and Election Period Queries
- Used `DATE_TRUNC` to aggregate spend and impression volume by month
  and by week, producing the time series data underlying the seasonal
  trend line on the dashboard
- Joined the primary ads table to the election enrichment table to
  group ad volume and spend by election type — confirming that the
  pre-election surge pattern held across all classified election
  periods in both 2018 and 2019
- Applied a window function cumulative sum partitioned by country and
  ordered by week to produce the running total spend series used in
  the Seasonal Influence trend line visualization

### Year-Over-Year Growth Queries
- Wrote a pivot-style query using conditional aggregation to place
  each organization's 2018 and 2019 spend side by side in a single
  result row, then computed the year-over-year change as a derived
  column — producing the underlying data for the slope chart
- Flagged any organization with at least one minor-targeted campaign
  using a max aggregation on the boolean flag so the minor targeting
  indicator carried through to the year-over-year comparison

### Spend Distribution Queries
- Used `PERCENTILE_CONT` to calculate the full spend distribution
  including the 25th, 50th, and 75th percentiles alongside the mean
  and max — confirming the heavily right-skewed distribution where the
  median of $169 sits far below the mean of $1,330 due to a small
  number of high-budget advertisers
- Queried the top 10 highest-spend ads with their full targeting
  profile including age bracket, minor targeting flag, criteria count,
  and spend-per-impression ratio to give a complete picture of what
  the highest-investment campaigns looked like

## Business Impact

### Addressing Structural Nulls Without Distorting Targeting Rates
- **Risk Prevented:** Imputing placeholder values into targeting fields
  that were legitimately null would have falsely inflated targeting usage
  rates — a null in `Gender` means gender targeting was not used, not
  that data is missing; filling it would have made the 10.59% gender
  targeting rate appear higher and misrepresented advertiser behavior
- **Analytical Impact:** By preserving nulls as meaningful absences and
  using binary flag columns for targeting presence/absence, the audience
  segmentation donut charts reflect the true rate at which each targeting
  dimension was actively chosen — giving regulators, researchers, and
  platforms an accurate picture of how targeting tools are actually used

### Standardizing Country Names
- **Risk Prevented:** The casing inconsistency across 29 countries would
  have split the same country into multiple dimension members in Tableau —
  the US appearing as both `united states` and `United states` would have
  divided the 1,667 US records, making the country bar chart show phantom
  entries and the Countries KPI overcount the number of distinct markets
- **Analytical Impact:** The corrected country dimension enables accurate
  cross-market spend comparisons, correct geographic filtering, and the
  reliable identification of the top five countries driving ad volume in
  the seasonal trend analysis

### Flagging Minor-Targeted Ads
- **Risk Prevented:** Without a structured minor-targeting flag, the age
  bracket data would have required manual inspection to identify which
  of the 102 unique age bracket values included individuals under 18 —
  making it impossible to systematically quantify the scale of
  minor-targeted political advertising
- **Ethical Impact:** The flag revealed that a significant share of
  advocacy ad spend was directed at users as young as 14 — a finding
  that would not have surfaced from raw data alone and that is central
  to the dashboard's ethical analysis and public accountability argument
- **Policy Impact:** This finding directly supports the conclusion that
  digital platforms need transparent, audience-centric targeting analysis
  to counteract algorithmic bias and protect young users from political
  and advocacy manipulation

### Validating the Location Targeting Three-Tier Structure
- **Risk Prevented:** Treating `Location` as a binary flag like the other
  seven targeting columns would have incorrectly collapsed granular
  targeting (metro area, zip code level) and broad targeting (country
  level) into the same category — understating how precisely some
  advertisers were able to reach specific local audiences
- **Analytical Impact:** Documenting the three-tier structure allowed
  the analysis to distinguish between advertisers running national
  awareness campaigns and those running hyper-local political ads
  targeting specific electoral districts — a meaningful difference
  for transparency and accountability reporting

### Enriching with Election Period Classification
- **Risk Prevented:** Without classifying ads by their election context,
  the seasonal trend spikes in the Ad Volume Trends chart would have
  appeared as unexplained anomalies — the data would have shown volume
  increases but provided no explanation for why they occurred
- **Analytical Impact:** The election type enrichment transformed the
  seasonal trend line from a descriptive chart into an explanatory one —
  confirming that ad volume increases are not random or organic but are
  directly triggered by approaching electoral events, which is the
  central finding of the seasonal analysis section

## Recommendations

### For Regulators and Policymakers
- **Establish a minimum age floor for political ad targeting** — the
  finding that campaigns actively targeted users as young as 14 years
  old with political and advocacy content demonstrates that self-reported
  transparency databases alone are insufficient; a hard regulatory floor
  prohibiting political targeting below age 18 is needed
- **Require standardized country and advertiser name formatting in
  transparency databases** — the casing inconsistencies found across
  29 countries would have made cross-country analysis unreliable without
  manual correction; standardized submission formats should be mandated
  at the platform level so public databases are analysis-ready
- **Mandate disclosure of targeting criteria counts** — the finding that
  90% of political ads used at least two targeting criteria confirms that
  micro-targeting is the norm, not the exception; regulators should
  require platforms to publish the number and type of targeting dimensions
  applied to every political ad, not just the ad creative

### For Snapchat and Digital Platforms
- **Implement automatic age-gating for political and advocacy ad
  categories** — given that 91.12% of all political ads used age-based
  targeting and many included minors in their brackets, platforms should
  enforce a system-level block preventing any political or advocacy ad
  from being served to users under 18, regardless of what the advertiser
  submits
- **Improve transparency database data quality at ingestion** — the null
  rates in fields like `Electoral District` (97.8% null) and
  `CandidateBallotInformation` (97.1% null) reflect optional fields that
  most advertisers skip; making these fields required for political ad
  categories would produce a far more complete and useful transparency
  record
- **Add pre-election surge monitoring** — the seasonal analysis confirmed
  that ad volume spikes before every major election across all five top
  countries; platforms should build automated monitoring that flags
  unusual volume increases in the 60-day window before scheduled
  elections and triggers enhanced review of new political ad submissions
  during those periods

### For Researchers and Civic Organizations
- **Use the minor-targeting flag as a replicable audit methodology** —
  the approach of scanning all age bracket values for lower bounds below
  18 and creating a structured flag column can be applied to any
  platform's political ad transparency database; this methodology should
  be standardized and applied to Facebook, Google, and TikTok ad
  libraries to enable cross-platform comparison of minor-targeting rates
- **Prioritize cross-market comparisons for advocacy categories** —
  the cross-market spend comparison revealed that gun safety, voter
  participation, and government organizations are the dominant spenders
  in both the US and UK; tracking how advocacy categories shift spend
  across election cycles would reveal whether specific issues receive
  coordinated international advertising pushes

## Key Insights
- **Scale:** 2,927 unique ads across 29 countries generated over 1.72
  billion impressions and $3.89 million in total verified spend
- **Targeting Intensity:** 90% of political ads used at least two
  targeting criteria simultaneously — age was the most common at 91.12%
  of all ads, followed by behavioral segments at 66.86%
- **Minor Exposure:** 37 distinct age brackets in the dataset included
  users under 18, with some campaigns starting as low as age 14 —
  75% of targeted advocacy spending reached younger demographics
- **Electoral Correlation:** Ad volume increased in all five top countries
  immediately before major political elections — confirming that
  Snapchat's political ad ecosystem is driven by electoral cycles rather
  than organic advertiser activity
- **Spend Concentration:** The top 10 spenders accounted for a
  disproportionate share of total spend — General Mills alone spent
  $509,621, representing 13.1% of all ad spend in the dataset
- **US Dominance:** The United States generated 1,667 of 2,927 total
  ad records (57%) and $2.26 million of $3.89 million in total spend
  (58%) — making it by far the largest market for Snapchat political
  advertising
