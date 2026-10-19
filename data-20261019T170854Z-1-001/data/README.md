# Data Directory & Team Handoff Specification

Both raw CSV inputs (`data/*.csv`) and processed integrated datasets (`data/processed/*.csv`) are excluded from Git version control via `.gitignore`.

To access the data locally:
1. **Raw Source CSVs (`data/`):** Available in the [team Google Drive project folder](https://drive.google.com/drive/folders/10ERrXETxYL9nknCJd9Ar7y88v1bNHgeq). Copy all 5 original CSV files into `data/`.
2. **Processed Integrated CSVs (`data/processed/`):**
   - **Local Generation (Available Immediately):** With the 5 raw CSVs in `data/`, execute [`notebooks/02_data_cleaning_integration.ipynb`](../notebooks/02_data_cleaning_integration.ipynb) to generate and verify all three files in `data/processed/`.
   - **Shared Google Drive Folder (Upload Pending):** Upload of the 3 processed CSV files (`listings_clean.csv`, `calendar_clean.csv`, `reviews_clean.csv`) to the [team Google Drive project folder](https://drive.google.com/drive/folders/10ERrXETxYL9nknCJd9Ar7y88v1bNHgeq) is pending as the final step of D09 handoff.

---

## 1. Raw Source Datasets (`data/`)

| File Name | Rows × Columns | Grain | Primary / Candidate Key | Notes |
| --- | --- | --- | --- | --- |
| `listings.csv` | `3,818 × 92` | One row per listing | `id` | Snapshot of Seattle listings (scraped `2016-01-04`); `neighbourhood_cleansed` has 87 categories (`0` missing). |
| `calendar.csv` | `1,393,570 × 4` | One row per listing-day | `(listing_id, date)` | Balanced panel (`3,818` listings × `365` calendar dates from `2016-01-04` to `2017-01-02`). |
| `reviews.csv` | `84,849 × 6` | One row per review | `id` | Historical reviews from `2009-06-07` to `2016-01-03` across `3,191` unique listings. |
| `seattle_holidays_events_2016.csv` | `19 × 5` | One row per event date | `date` | 19 unique dates (`2016-01-01` to `2017-01-02`); 18 dates fall within the calendar window. |
| `seattle_walkscore_raw.csv` | `80 × 7` | One row per neighborhood | `neighborhood` | Walk, Transit, and Bike Scores across 80 Seattle neighborhoods (`73` exact name matches with `neighbourhood_cleansed`). |

---

## 2. Processed Integrated Datasets (`data/processed/`)

Generated and validated in [`notebooks/02_data_cleaning_integration.ipynb`](../notebooks/02_data_cleaning_integration.ipynb) (`0` rows dropped, `0` baseline columns dropped, `0` missing values imputed):

| File Name | Rows × Columns | Grain | Primary Key | Summary of Contents |
| --- | --- | --- | --- | --- |
| `listings_clean.csv` | `3,818 × 102` | One row per listing | `id` | All `92` cleaned baseline columns + `5` Walk Score columns + `5` listing-level review summary columns. |
| `calendar_clean.csv` | `1,393,570 × 10` | One row per listing-day | `(listing_id, date)` | All `4` cleaned baseline calendar columns + `6` date-level holiday and event indicator columns. |
| `reviews_clean.csv` | `84,849 × 6` | One row per review | `id` | Cleaned review records (`date` parsed to ISO `YYYY-MM-DD`; `18` missing `comments` preserved as `NaN`; `0` whitespace-only strings). |

---

## 3. Dictionary of Added Features

### 3.1 Added Columns in `listings_clean.csv` (10 Columns)

| Column Name | Source & Join Key | Description | Dtype & Unit / Observed Range | Missingness & Analytical Interpretation |
| --- | --- | --- | --- | --- |
| `walk_score` | `seattle_walkscore_raw.csv` via exact match `listings.neighbourhood_cleansed == walkscore.neighborhood` | Neighborhood Walk Score mobility index | `Int64` (index points; Observed Range: `33–98`) | `1,085` missing (`28.42%`) across `14` cleansed neighborhoods with no confirmed correspondence (`<NA>`). |
| `transit_score` | `seattle_walkscore_raw.csv` via exact match `listings.neighbourhood_cleansed == walkscore.neighborhood` | Neighborhood Transit Score index | `Int64` (index points; Observed Range: `33–100`) | `1,085` missing (`28.42%`) across `14` unmapped neighborhoods (`<NA>`). |
| `bike_score` | `seattle_walkscore_raw.csv` via exact match `listings.neighbourhood_cleansed == walkscore.neighborhood` | Neighborhood Bike Score index | `Int64` (index points; Observed Range: `28–89`) | `1,085` missing (`28.42%`) across `14` unmapped neighborhoods (`<NA>`). |
| `ws_population` | `seattle_walkscore_raw.csv` via exact match `listings.neighbourhood_cleansed == walkscore.neighborhood` | Neighborhood population reported in Walk Score table | `Int64` (residents; Observed Range: `529–26,712`) | `1,085` missing (`28.42%`) across `14` unmapped neighborhoods (`<NA>`). |
| `ws_rank` | `seattle_walkscore_raw.csv` via exact match `listings.neighbourhood_cleansed == walkscore.neighborhood` | Neighborhood walkability rank in Walk Score table | `Int64` (rank order; Observed Range: `1–79`) | `1,085` missing (`28.42%`) across `14` unmapped neighborhoods (`<NA>`). |
| `rev_count_from_reviews` | `reviews.csv` aggregated via `groupby(reviews.listing_id)` and joined on `listings.id == reviews.listing_id` | Total historical review records for the listing in `reviews.csv` | `Int64` (count of reviews; Observed Range: `1–474`) | `627` missing (`16.42%`), corresponding to the `627` unreviewed listings (`number_of_reviews == 0`). |
| `rev_first_date_from_reviews` | `reviews.csv` aggregated via `groupby(reviews.listing_id)` and joined on `listings.id == reviews.listing_id` | Earliest review date for the listing in `reviews.csv` | `datetime64[ns]` (ISO date; Observed Range: `2009-06-07` to `2016-01-03`) | `627` missing (`16.42%`) for unreviewed listings (`NaT`). |
| `rev_last_date_from_reviews` | `reviews.csv` aggregated via `groupby(reviews.listing_id)` and joined on `listings.id == reviews.listing_id` | Latest review date for the listing in `reviews.csv` | `datetime64[ns]` (ISO date; Observed Range: `2011-09-17` to `2016-01-03`) | `627` missing (`16.42%`) for unreviewed listings (`NaT`). |
| `rev_unique_reviewers` | `reviews.csv` aggregated via `groupby(reviews.listing_id)` and joined on `listings.id == reviews.listing_id` | Number of distinct `reviewer_id` values for the listing | `Int64` (count of unique reviewers; Observed Range: `1–463`) | `627` missing (`16.42%`) for unreviewed listings (`<NA>`). |
| `rev_avg_comment_length` | `reviews.csv` aggregated via `groupby(reviews.listing_id)` and joined on `listings.id == reviews.listing_id` | Mean character length of non-null review `comments` for the listing | `float64` (characters; Observed Range: `7.00–1,875.00`) | `627` missing (`16.42%`) for unreviewed listings (`NaN`). |

### 3.2 Added Columns in `calendar_clean.csv` (6 Columns)

| Column Name | Source & Join Key | Description | Dtype & Unit / Observed Range | Missingness & Analytical Interpretation |
| --- | --- | --- | --- | --- |
| `event_record_count` | `seattle_holidays_events_2016.csv` pre-aggregated by `date` (`calendar.date == holidays.date`) | Number of source records in the holiday/event table matching `date` | `Int64` (count of records; Observed Range: `0–1`) | `0` missing (`0.00%`); `0` on `347` unlisted calendar dates (`1,324,846` listing-days), `1` on `18` matched dates (`68,724` listing-days). |
| `event_names` | `seattle_holidays_events_2016.csv` pre-aggregated by `date` (`calendar.date == holidays.date`) | Name of holiday or city event on `date` | `object` / string (`15` distinct event name labels across the `18` matched calendar dates) | `1,324,846` missing (`95.07%`), representing the `347` calendar dates with no record in the event source (not confirmed event-free days). |
| `event_types` | `seattle_holidays_events_2016.csv` pre-aggregated by `date` (`calendar.date == holidays.date`) | Category label from event source (`Federal Holiday`, `Seattle Event`, `Seattle Festival`, `Major Convention & Festival`, `Federal Holiday & Convention`) | `object` / string (`5` distinct category labels) | `1,324,846` missing (`95.07%`) on the `347` calendar dates with no record in the event source. |
| `event_neighborhoods` | `seattle_holidays_events_2016.csv` pre-aggregated by `date` (`calendar.date == holidays.date`) | Geographic focus label from event source (`Citywide`, `Capitol Hill`, etc.) | `object` / string (`6` distinct focus labels) | `1,324,846` missing (`95.07%`) on the `347` calendar dates with no record in the event source. |
| `is_holiday` | Derived from `event_types` on `date` | Indicator for federal holiday dates (including the joint `2016-09-05` Labor Day / PAX West record) | `boolean` (`True` on `10` dates / `38,180` listing-days; `False` on `355` dates / `1,355,390` listing-days) | `0` missing (`0.00%`). |
| `is_event` | Derived from `event_record_count > 0` on `date` | Indicator for the presence of **any holiday or event record** in `seattle_holidays_events_2016.csv` on `date` (to isolate the `8` non-holiday event dates / `30,544` listing-days, filter `is_event & (~is_holiday)`) | `boolean` (`True` on `18` matched dates / `68,724` listing-days; `False` on `347` unlisted dates / `1,324,846` listing-days) | `0` missing (`0.00%`). |

---

## 4. Reloading & Type Restoration Specification

Because standard CSV files do not preserve pandas nullable types (`boolean`, `Int64`) or `datetime64[ns]`, use the following loader snippet in downstream modeling and NLP notebooks.

> **Working Directory Note:** The path `Path("data/processed")` assumes execution from the **project root directory (`seattle-airbnb-ml/`)**. When running from inside `notebooks/`, the fallback `Path("../data/processed")` automatically resolves the directory.

```python
import pandas as pd
from pathlib import Path

PROCESSED_DIR = Path("data/processed")
if not PROCESSED_DIR.exists():
    PROCESSED_DIR = Path("../data/processed")

# 1. Listings Reload (3,818 rows × 102 columns)
DATE_COLS_LISTINGS = [
    'last_scraped', 'host_since', 'calendar_last_scraped',
    'first_review', 'last_review',
    'rev_first_date_from_reviews', 'rev_last_date_from_reviews'
]
BOOL_COLS_LISTINGS = [
    'host_is_superhost', 'host_has_profile_pic', 'host_identity_verified',
    'is_location_exact', 'has_availability', 'requires_license',
    'instant_bookable', 'require_guest_profile_picture', 'require_guest_phone_verification'
]
INT_COLS_LISTINGS = [
    'walk_score', 'transit_score', 'bike_score', 'ws_population', 'ws_rank',
    'rev_count_from_reviews', 'rev_unique_reviewers'
]

dtypes_listings = {col: 'boolean' for col in BOOL_COLS_LISTINGS}
dtypes_listings.update({col: 'Int64' for col in INT_COLS_LISTINGS})

df_listings = pd.read_csv(
    PROCESSED_DIR / 'listings_clean.csv',
    parse_dates=DATE_COLS_LISTINGS,
    dtype=dtypes_listings
)

# 2. Calendar Reload (1,393,570 rows × 10 columns)
DATE_COLS_CALENDAR = ['date']
BOOL_COLS_CALENDAR = ['available', 'is_holiday', 'is_event']
INT_COLS_CALENDAR = ['event_record_count']

dtypes_calendar = {col: 'boolean' for col in BOOL_COLS_CALENDAR}
dtypes_calendar.update({col: 'Int64' for col in INT_COLS_CALENDAR})

df_calendar = pd.read_csv(
    PROCESSED_DIR / 'calendar_clean.csv',
    parse_dates=DATE_COLS_CALENDAR,
    dtype=dtypes_calendar
)

# 3. Reviews Reload (84,849 rows × 6 columns)
df_reviews = pd.read_csv(
    PROCESSED_DIR / 'reviews_clean.csv',
    parse_dates=['date']
)
```

---

## 5. Downstream Workstream Notes (Marvin Pearson & Rett Kinsey)

- **Exclusion Candidates in `listings_clean.csv`:**
  - `license` (`3,818` missing, `100.00%`) and `square_feet` (`3,721` missing, `97.46%`) are retained in the CSV for completeness but should be excluded from predictive feature sets.
- **Un-Imputed Missingness Patterns:**
  - `walk_score`, `transit_score`, and `bike_score`: missing for `1,085` listings (`28.42%`) across `14` unmapped neighborhoods.
  - `review_scores_rating`: missing for `647` listings (`16.95%`), consisting of `627` unreviewed listings (`number_of_reviews == 0`) and `20` reviewed listings (`number_of_reviews > 0`) without a recorded rating. Six sub-scores have `651–658` missing values (`17.05%–17.23%`).
  - `host_response_time` and `host_response_rate`: missing for `523` listings (`13.70%`); `host_acceptance_rate` is missing for `773` listings (`20.25%`); `host_is_superhost` and `host_identity_verified` are missing for `2` listings (`0.05%`).
  - `reviews_clean.csv` (`comments`): `84,831` non-null strings and `18` missing values (`0.02%`; `0` whitespace-only strings) across `3,191` reviewed listings (`627` listings in `listings_clean.csv` have `0` reviews).
  - Any imputation, scaling, or encoding in modeling workflows must be fit strictly on the training split (`fit on train only`).
- **Checking for Data Leakage After Task Definition:**
  - `listings_clean.csv` represents a static snapshot as of `2016-01-04` alongside cumulative historical review summaries (`2009-06-07` to `2016-01-03`).
  - `calendar_clean.csv` represents forward-looking availability and advertised prices from `2016-01-04` to `2017-01-02` (`available == False` is an occupancy proxy, not confirmed bookings, and `price` is recorded only when `available == True`).
  - Data leakage is a modeling risk that must be evaluated once the specific prediction target, observation cutoff, and train/test split are formally defined.
