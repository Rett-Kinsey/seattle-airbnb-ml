# Seattle Airbnb Project — Issues, Obstacles & Limitations Log

**Course:** DSBA/HCIP 6156 Applied Machine Learning | Fall 2026  
**Document Purpose:** Living repository of empirical data quality issues, methodological obstacles, and analytical risks. Feeds directly into **Section 4 ("Any issues or obstacles encountered")** of the Midterm Progress Report (due Oct 21) and the Final Project Report.

> **Methodological Standard:** Every documented issue follows a strict four-part schema:  
> **1. Observed Empirical Fact** $\rightarrow$ **2. Potential Consequence / Risk** $\rightarrow$ **3. Candidate Analytical Approach** $\rightarrow$ **4. Verification Status**

---

## Executive Summary of Issues

```
+---------------------------------------------------------------------------------------------------+
|                                   IDENTIFIED PROJECT OBSTACLES                                    |
+---------------------------------------------------------------------------------------------------+
| 1. DATA QUALITY & PROFILING                                                                       |
|    - Near-empty features: license (100% NaN), square_feet (97.46% NaN).                            |
|    - Optional fee missingness: monthly_price (60.3%), security_deposit (51.1%), cleaning_fee (27%). |
|    - String-encoded numericals: currency ($ and ,), percentages (%), booleans ('t'/'f').          |
+---------------------------------------------------------------------------------------------------+
| 2. STRUCTURAL vs. INCIDENTAL MISSINGNESS (REVIEWS & RATINGS)                                      |
|    - 627 listings (16.42%) have zero reviews -> 100% NaN in review dates, frequency, and ratings.  |
|    - Incidental missingness: 20-31 listings WITH reviews also lack category rating scores.        |
|    - Root causes unproven; listwise deletion removes 16.42% - 17.29% depending on fields.         |
+---------------------------------------------------------------------------------------------------+
| 3. METHODOLOGICAL LIMITATION: OCCUPANCY PROXY                                                     |
|    - calendar.csv available == 'f' (32.94% of days) is an OCCUPANCY PROXY, not confirmed bookings.|
|    - Host calendar blocking (maintenance, personal stay, minimum stay) is indistinguishable.      |
|    - 100% of price values are missing when available == 'f' (no observed transaction prices).      |
+---------------------------------------------------------------------------------------------------+
| 4. TAXONOMY & SPATIAL INTEGRATION MISMATCH (WALK SCORE)                                           |
|    - listings.csv has 87 cleansed neighborhoods; Walk Score has 80 neighborhoods.                 |
|    - 14 neighborhoods in listings.csv (1,085 listings, 28.42% of dataset) do not match directly.   |
|    - Proposed geographic mappings are unverified hypotheses; boundary equivalence unproven.       |
+---------------------------------------------------------------------------------------------------+
| 5. TEMPORAL, SPATIAL & COMPLETENESS LIMITATIONS OF EVENTS (HOLIDAYS & EVENTS)                     |
|    - Temporal truncation: 2016-01-01 (New Year's Day) precedes calendar coverage (2016-01-04).    |
|    - Geographic mismatch: 0 matches with neighbourhood_cleansed; only 'Capitol Hill' matches     |
|      neighbourhood_group_cleansed (and string equality does not prove identical boundaries).       |
|    - Unverified exhaustiveness: 19 curated dates have source attribution strings, but are not an  |
|      independently verified or complete census of all 2016 Seattle events.                        |
|    - Non-causality: event presence cannot be presumed to correlate with or drive demand/pricing.  |
+---------------------------------------------------------------------------------------------------+
| 6. GRAIN MISMATCH & LEAKAGE RISKS                                                                 |
|    - Grain differences: listing (3.8k), listing-day (1.39M), review (84.8k), neighborhood (80),   |
|      event date (19).                                                                             |
|    - Merging detail tables (Calendar + Reviews) solely on listing_id creates Cartesian explosion. |
|    - Future dates in calendar reflect forward availability at scrape time, not realized demand;   |
|      leakage risks must be assessed strictly against the defined prediction target.               |
+---------------------------------------------------------------------------------------------------+
| 7. WORKFLOW & REPRODUCIBILITY INCIDENTS                                                            |
|    - Output loss during rollback/replay in Cells 2, 4, 6, 7 detected by comparative audit.         |
|    - Exact original code/outputs restored from transcript; verified and closed.                    |
+---------------------------------------------------------------------------------------------------+
```

---

## Detailed Issue Breakdown

### 1. Data Quality & Missingness Obstacles

#### 1.1 Unusable Columns (Severe Missingness: `license`, `square_feet`)
* **Observed Empirical Fact:** `license` contains $3\,818$ missing values ($100.00\%$). `square_feet` contains $3\,721$ missing values ($97.46\%$).
* **Potential Consequence / Risk:** An extremely high proportion of missing values ($>97\%$) limits the methodological justification for statistical imputation; dropping these features is considered as one candidate option during Data Preparation.
* **Candidate Analytical Approach:** Evaluate dropping both columns during the Data Preparation phase.
* **Verification Status:** Confirmed and audited in Phase D2.1 (`02_data_cleaning_integration.ipynb`). In accordance with project boundaries, both columns are retained in `listings_clean` as documented exclusion candidates without premature automatic dropping. Retained for now; final exclusion decision will be documented during feature preparation.

#### 1.2 Missingness in Optional Commercial Terms (`monthly_price`, `security_deposit`, `weekly_price`, `cleaning_fee`)
* **Observed Empirical Fact:** High proportion of missing values across optional pricing terms: `monthly_price` ($60.27\%$), `security_deposit` ($51.13\%$), `weekly_price` ($47.38\%$), `cleaning_fee` ($26.98\%$).
* **Potential Consequence / Risk:** The true underlying semantic meaning of missingness is unrecorded in the dataset: a missing fee could represent a $\$0$ charge (no fee required), a host omission, or unavailable information.
* **Candidate Analytical Approach:** Test competing hypotheses during Data Preparation: (a) imputing $\$0$ under the assumption that omitted fees mean no charge, versus (b) creating explicit binary indicator features (`has_cleaning_fee = 0/1`, `has_security_deposit = 0/1`).
* **Verification Status:** Unverified in source data. Imputation with $\$0$ is an analytical option for empirical comparison, not an established factual meaning.

#### 1.3 Internal Taxonomy Inconsistency in `listings.csv`
* **Observed Empirical Fact:** Raw `neighbourhood` is missing in $416$ listings ($10.90\%$), whereas `neighbourhood_cleansed` and `neighbourhood_group_cleansed` are $100.00\%$ complete ($0$ missing values).
* **Potential Consequence / Risk:** Using raw `neighbourhood` would unnecessarily inject missing values into spatial groupings.
* **Candidate Analytical Approach:** Base spatial features and joins exclusively on `neighbourhood_cleansed` and `neighbourhood_group_cleansed`.
* **Verification Status:** Factually confirmed by missingness audit of `listings.csv`.

---

### 2. The "Zero-Reviews" Structural Missingness

* **Observed Empirical Fact:**
  * Exactly $627$ listings ($16.42\%$) have `number_of_reviews == 0`.
  * For all $627$, $100\%$ of review dates (`first_review`, `last_review`), `reviews_per_month`, and all $7$ rating scores (`review_scores_rating`, `accuracy`, `cleanliness`, `checkin`, `communication`, `location`, `value`) are missing.
  * In addition, among the $3\,191$ listings *with* reviews, between $20$ and $31$ listings lack category rating scores (incidental missingness).
* **Potential Consequence / Risk:**
  * The proportion of data lost through listwise deletion depends directly on the specific review fields required by the model:
    - Dropping listings with zero reviews removes **$627$ listings ($16.42\%$)**.
    - Dropping listings missing the overall `review_scores_rating` removes **$647$ listings ($16.95\%$)**.
    - Dropping listings missing any of the seven rating categories removes **$660$ listings ($17.29\%$)**.
  * Eliminating these listings distorts the market sample by systematically removing all unreviewed properties and properties with incomplete ratings.
* **Candidate Analytical Approach:**
  * Create an explicit binary indicator (`has_reviews = 0/1`).
  * Compare imputation strategies for missing ratings (e.g., median rating of reviewed listings versus a dedicated category/indicator) during feature engineering, evaluating impact on model performance.
* **Verification Status:**
  * The absence of reviews does **not** prove that a listing is a new market entrant; it could also reflect lack of bookings, unreviewed stays, or listing inactivity.
  * The root cause of zero reviews remains an unverified hypothesis. Imputation and indicator encoding are candidate options under evaluation.

---

### 3. Methodological Limitation: The Occupancy Proxy

* **Observed Empirical Fact:**
  * In `calendar.csv`, $459\,028$ records ($32.94\%$) are flagged as `available == 'f'`.
  * Exactly $459\,028$ price values are missing ($100\%$ null when `available == 'f'`).
* **Potential Consequence / Risk:**
  * Public scrape data does not record completed booking transactions or confirmed revenue.
  * A day flagged `available == 'f'` can represent:
    1. A booked guest stay.
    2. Host calendar blocking for personal use, holidays, or maintenance.
    3. Host minimum-stay constraints preventing single-day booking.
  * Actual transaction prices on booked days are unobserved.
* **Candidate Analytical Approach:**
  * Define availability-derived metrics strictly as an **Occupancy Proxy (share of unavailable days)**, never as verified bookings or true occupancy.
  * Do not compute or report host revenue estimates as ground truth.
* **Verification Status:**
  * Methodological constraint verified by data schema and Airbnb scrape design.

---

### 4. Spatial Integration & Taxonomy Mismatch (Walk Score)

* **Observed Empirical Fact:**
  * `listings.csv` records $87$ unique neighborhood names in `neighbourhood_cleansed`.
  * `seattle_walkscore_raw.csv` records $80$ neighborhood names.
  * Exactly $73$ names match character-for-character (covering $2\,733$ listings, $71.58\%$).
  * Exactly **$14$ neighborhoods in `listings.csv` do not match directly**, representing **$1\,085$ listings ($28.42\%$ of the dataset)**:

| Unmatched Listings Neighborhood | Listing Count | % of Dataset | Walk Score Status |
|---|---|---|---|
| **Broadway** | 397 | 10.40% | No confirmed correspondence |
| **Minor** | 135 | 3.54% | No confirmed correspondence |
| **Stevens** | 119 | 3.12% | No confirmed correspondence |
| **Central Business District** | 103 | 2.70% | No confirmed correspondence |
| **Adams** | 70 | 1.83% | No confirmed correspondence |
| **Mann** | 67 | 1.75% | No confirmed correspondence |
| **West Woodland** | 64 | 1.68% | No confirmed correspondence |
| **Alki** | 42 | 1.10% | No confirmed correspondence |
| **Pike-Market** | 28 | 0.73% | No confirmed correspondence |
| **Wedgwood** | 18 | 0.47% | No confirmed correspondence |
| **Harrison/Denny-Blaine** | 14 | 0.37% | No confirmed correspondence |
| **North Beach/Blue Ridge** | 14 | 0.37% | No confirmed correspondence |
| **Cedar Park** | 8 | 0.21% | No confirmed correspondence |
| **Holly Park** | 6 | 0.16% | No confirmed correspondence |
| **TOTAL UNMATCHED** | **1,085** | **28.42%** | **No confirmed correspondence (0 expanded matches)** |

* **Potential Consequence / Risk:**
  * A standard join leaves $1\,085$ listings ($28.42\%$) with `NaN` for mobility metrics.
  * Direct character string equality for the $73$ matching names does **not** prove identical geographic polygon boundaries between WalkScore.com and City of Seattle / Airbnb administrative definitions.
* **Candidate Analytical Approach:**
  * Strict methodological standard: do not perform speculative candidate substitutions without GIS polygon boundary proof.
  * Retain all $1\,085$ unmapped listings strictly as missing (`NaN`); avoid synthetic mean/median imputation.
* **Verification Status:** Formally concluded in Phase D2.2 (`02_data_cleaning_integration.ipynb`). Additional geographically confirmed correspondences = 0. All 14 non-matching neighborhoods ($1\,085$ listings, $28.42\%$) remain classified as **"No confirmed correspondence"**. The crosswalk dictionary is empty (`confirmed_crosswalk = {}`). Baseline coverage remains strictly at $2\,733$ listings ($71.58\%$) based solely on exact string matches (geographic boundary equivalence unverified). Coverage was not expanded. Zero synthetic imputation applied.

---

### 5. Temporal, Spatial & Completeness Limitations of Events (`seattle_holidays_events_2016.csv`)

* **Observed Empirical Fact:**
  * Table contains $19$ rows and $19$ unique dates ($2016-01-01$ to $2017-01-02$).
  * Date `2016-01-01` (New Year's Day) occurs before the `calendar.csv` start date ($2016-01-04$).
  * Exactly $18$ dates match calendar dates, corresponding to $68\,724$ calendar rows ($4.93\%$) across all $3\,818$ listings in our dataset.
  * Exactly $347$ calendar dates have no record in this table ($95.07\%$ of calendar days).
  * `neighborhood_focus` contains $6$ distinct values: $0$ match `neighbourhood_cleansed`; only $1$ (`Capitol Hill`) matches `neighbourhood_group_cleansed`.
  * The `source` column contains attribution strings (`U.S. OPM`, archive names) for all $19$ rows ($100\%$ completeness within this file).
* **Potential Consequence / Risk:**
  * `2016-01-01` cannot be linked to calendar records.
  * String matching for `Capitol Hill` does not prove identical geographic boundaries between event locations and administrative neighborhood groups.
  * The $347$ calendar dates without records cannot be assumed to be event-free days in Seattle.
  * The `source` strings document data entry provenance in the CSV, but do not represent independent validation against primary archives or guarantee exhaustive coverage of all 2016 Seattle events.
  * Assuming that event presence drives price or availability proxy variations would introduce unjustified causal claims.
* **Candidate Analytical Approach:**
  * Perform a left join from `calendar.csv` on `date`, leaving dates without matches coded as "no event recorded in this source".
  * Test citywide vs. localized event indicator flags during feature engineering.
  * Formally evaluate the statistical relationship between event dates and price/availability proxy variations during EDA, rather than assuming causality.
* **Verification Status:**
  * Temporal mismatch and taxonomy non-alignment confirmed by code.
  * Event exhaustiveness and primary source validity are unverified.
  * Any event impact on demand or pricing remains an untested hypothesis.

---

### 6. Architectural & Leakage Considerations

#### 6.1 Granularity Differences & Join Multiplicity
* **Observed Empirical Fact:** Each dataset has a distinct grain: Listings ($3\,818$ listings), Calendar ($1\,393\,570$ listing-days), Reviews ($84\,849$ reviews), Walk Score ($80$ neighborhoods), Holidays/Events ($19$ dates).
* **Potential Consequence / Risk:**
  * Merging Listings with Calendar or Reviews individually represents a standard one-to-many relationship ($1:365$ or $1:N$), which duplicates listing attributes across detail records.
  * Merging unaggregated Calendar and Reviews tables together solely on `listing_id` (without aligning on date or aggregating) would create a Cartesian product (many-to-many explosion).
* **Candidate Analytical Approach:**
  * Aggregate Calendar metrics (e.g., mean price, share of unavailable days) and Reviews metrics (e.g., total review count, sentiment summary) to the `listing_id` grain before merging into the tabular modeling dataset.
* **Verification Status:**
  * Confirmed by primary key and grain inspection.

#### 6.2 Forward Calendar Dates & Target Leakage Assessment
* **Observed Empirical Fact:** `calendar.csv` contains dates spanning 365 days forward from the snapshot date ($2016-01-04$).
* **Potential Consequence / Risk:**
  * Calendar records reflect forward host availability and pricing settings captured at the snapshot time ($2016-01-04$), not realized future transactions.
  * If the predictive task is predicting static listing price at snapshot time, using forward calendar metrics as predictors risks lookahead leakage if those metrics reflect future market adjustments.
  * If the task is dynamic demand/occupancy prediction across time, evaluation must use a chronological train/test split rather than random K-Fold cross-validation.
* **Candidate Analytical Approach:**
  * Explicitly define the predictive target before feature selection.
  * For static price prediction, rely on historical listing attributes and historical reviews.
  * For dynamic calendar modeling, design a chronological split (e.g., train on Q1–Q3 2016, evaluate on Q4 2016).
* **Verification Status:**
  * Leakage risk depends on the specific modeling objective and must be formally assessed when formulating prediction targets.

#### 6.3 Phase D2.3 Dataset Assembly & Integration Architecture
* **Observed Empirical Fact:** In Phase D2.3 (`02_data_cleaning_integration.ipynb`), two canonical analytical datasets were assembled in memory:
  1. `df_listings_clean` ($3\,818$ rows $\times$ $102$ columns): integrates cleaned property attributes with Walk Score mobility metrics matched by exact name string equality ($73$ exact string matches, $2\,733$ listings; geographic boundary equivalence is unverified; $14$ unmapped neighborhoods, $1\,085$ listings with `<NA>`) and listing-level review summary metrics from `reviews.csv` ($3\,191$ enriched listings, $627$ unreviewed listings with `<NA>`).
  2. `df_calendar_clean` ($1\,393\,570$ rows $\times$ $10$ columns): integrates daily calendar records across the full calendar span ($2016-01-04$ to $2017-01-02$) with daily holiday and event indicators pre-aggregated from `seattle_holidays_events_2016.csv`.
* **Potential Consequence / Risk:**
  * In the current `seattle_holidays_events_2016.csv`, all 19 rows have unique dates (with Labor Day and PAX West combined into a single record on 2016-09-05). However, if an event source records multiple distinct rows for concurrent events on the same date, merging directly without prior date-level aggregation would cause Cartesian row expansion in the 1.39 million row calendar.
  * Adding prospective calendar aggregates (spanning $2016-01-04$ to $2017-01-02$) into static listing records may introduce lookahead bias and target leakage depending on the specific modeling objective and observation cutoff (e.g., if predicting listing success or price established at the January 2016 snapshot).
  * Dogmatic claims of "zero leakage" are methodologically invalid without a defined ML prediction task, observation horizon, and feature cutoff date.

  * Misinterpreting `is_event == False` as proof of zero civic/community activity introduces false negative bias regarding real-world Seattle events.
* **Candidate Analytical Approach:**
  * Pre-aggregate events strictly to one row per unique date (`groupby('date')`), concatenating event names/types and generating integer count flags (`event_record_count`, counting source records per date) and boolean indicators (`is_event`, `is_holiday`).
  * Explicit definition of event indicators: `is_event` strictly denotes the presence of a record in `seattle_holidays_events_2016.csv`; `False` does not prove the absence of real-world events.
  * Preserve explicit timestamps across all records and keep the calendar-day dataset compact (no static listing attributes duplicated across 1.39 million rows).
  * Isolate forward calendar outcomes from static listing metadata, allowing downstream modeling pipelines (Marvin Pearson) to establish task-specific prediction cutoffs.
  * Preserve raw review comments in `reviews_clean` independently for NLP text and topic modeling (Rett Kinsey).
* **Verification Status:**
  * Pre-aggregation confirmed: exactly 19 unique event dates in source; 18 dates fall within the calendar span ($68\,724$ calendar rows, $4.93\%$), 1 date precedes calendar start ($2016-01-01$).
  * Zero Cartesian duplication: `(listing_id, date)` certified unique across all $1\,393\,570$ rows.
#### 6.4 Phase D2.4 Integration Validation & Integrity Certification
* **Observed Empirical Fact:** In Phase D2.4 (`02_data_cleaning_integration.ipynb`), five programmatic verification checks were executed across the assembled datasets:
  1. *Rows & Keys:* Dimensions confirmed (`df_listings_clean`: $3\,818 \times 102$, `df_calendar_clean`: $1\,393\,570 \times 10$, `reviews_clean`: $84\,849 \times 6$); primary keys unique and non-null across all three tables (`id` in listings, `(listing_id, date)` in calendar, `id` in reviews); $0$ orphan records; exact 365-day balance per listing.
  2. *Data Preservation:* Original cleaned columns ($92$ in listings, $4$ in calendar) retain identical element-wise values before and after joins (`equals == True`).
  3. *Walk Score Fidelity:* $2\,733$ matched listings have scores matching the source table with $100\%$ fidelity; exactly $1\,085$ unmapped listings remain strictly `<NA>` across all Walk Score fields.
  4. *Reviews Concordance:* Split audit confirmed $3\,191$ reviewed listings match baseline counts and first/last dates exactly; $627$ unreviewed listings have strictly `<NA>` across all new review features.
  5. *Calendar & Events Alignment:* $365$-day timeline ($2016-01-04$ to $2017-01-02$), element-wise date alignment confirming all $68\,724$ event rows ($18 \times 3\,818$) and $38\,180$ holiday rows ($10 \times 3\,818$) match exact source dates ($0$ date misalignments), and $100\%$ alignment between missing prices and unavailable dates ($459\,028$ rows).
* **Potential Consequence / Risk:** Undetected join distortion, accidental Cartesian duplication, or silent type coercion could corrupt downstream exploratory data analysis and predictive modeling.
* **Candidate Analytical Approach:** Enforce strict assertions and compile a single compact summary validation table in the notebook certifying all five checks with observed values, expected values, and `PASS` status.
* **Verification Status:** All five checks verified and certified `PASS` in memory and saved to notebook cell outputs. Dataset serialization executed in Phase D2.5.

#### 6.5 Phase D2.5 Data Export & Type Restoration Protocol
* **Observed Empirical Fact:** In Phase D2.5 (`02_data_cleaning_integration.ipynb`), three canonical datasets were serialized to `data/processed/` in standard CSV format (`index=False`):
  1. `listings_clean.csv` ($3\,818$ rows $\times$ $102$ columns, $16.42$ MB): canonical listing-level dataset enriched with Walk Score and review summary features.
  2. `calendar_clean.csv` ($1\,393\,570$ rows $\times$ $10$ columns, $63.70$ MB): panel listing-day dataset with pre-aggregated event/holiday indicators.
  3. `reviews_clean.csv` ($84\,849$ rows $\times$ $6$ columns, $34.90$ MB): clean review table preserved for NLP modeling.
* **Potential Consequence / Risk:**
  * Plain CSV does not natively store pandas data types. When reloaded in downstream EDA or modeling notebooks, dates revert to object strings, booleans risk improper string interpretation, and nullable integers (`Int64`) with `<NA>` default to `float64`.
  * Naive row counting on CSV files (e.g., via simple line splitting) miscounts lines due to legitimate multiline review comments.
* **Candidate Analytical Approach:**
  * Perform full in-memory readback verification using pandas `pd.read_csv`, passing explicit `parse_dates` lists and read-time boolean mappings `dtype={col: 'boolean'}` rather than post-hoc `.astype('boolean')` on strings.
  * Verify dataset shapes, column orders, primary keys, 100% exact null position masks across every column, full DataFrame element-wise equality (`assert_frame_equal`), and complete comment text preservation across all $84\,849$ reviews without row subsetting.
  * Document the canonical reload schema directly in the notebook documentation.
* **Verification Status:**
  * Full readback verified: $100\%$ match on shapes, column ordering, unique primary keys, complete null masks, and all $84\,849$ review comments without truncation.
  * Full DataFrame equality verified across all three datasets via `assert_frame_equal` accounting for chosen types and floating-point tolerances.
  * Status certified: "Export completed; readback checks passed".
  * Raw source files in `data/` remain untouched; `data/processed/` is excluded from git tracking via `.gitignore`.

---

### 7. Workflow & Reproducibility Incidents

#### 7.1 Loss of Executed Cell Outputs During Working-Tree Rollback and Replay
* **Observed Empirical Fact:** During the addition of Section 3.5, a working-tree rollback (`git checkout`) followed by sequential execution of historical scratch scripts resulted in the loss of executed cell outputs in Cells 2 (Setup), 4 (Data Inventory), 6 (Listings Load), and 7 (Listings Exploration), and altered the code of Cell 7. This regression was detected via independent comparative audit against a preserved notebook copy.
* **Potential Consequence / Risk:** While the underlying computation code was sound, the missing outputs broke notebook self-containment and reproducibility, preventing an evaluator from inspecting listings data profiling results without re-executing earlier cells.
* **Candidate Analytical Approach:** Systematically restore the exact original code and outputs of Cells 2, 4, 6, and 7 from the verified historical transcript, maintain Section 3.5 intact, and incorporate explicit verification of listwise deletion impact (627, 647, 660 listings) directly inside the notebook.
* **Verification Status:** Resolved. Cell 7 source restored to `print(listings.dtypes.to_string())` providing complete 92-line type output without truncation; substantive content and outputs in Cells 2, 4, 6, and 7 confirmed matching the verified baseline copy (with minor differences only in blank line spacing in Cell 7). File sizes confirmed from data: `listings.csv` 16.87 MB, `calendar.csv` 36.65 MB, `reviews.csv` 36.59 MB, `seattle_walkscore_raw.csv` 0.01 MB (6,427 bytes), and `seattle_holidays_events_2016.csv` 0.00 MB (exact size: 1,697 bytes, ~1.70 KB; 0.00 MB reflects two-decimal rounding).
