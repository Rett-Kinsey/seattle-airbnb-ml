# Seattle Airbnb — Market Dynamics & Listing Success Prediction

**UNC Charlotte · DSBA 6156 Applied Machine Learning · Fall 2026**

Team project investigating Seattle Airbnb listings, price and calendar dynamics, neighborhood context, and review text. The aim is to understand market patterns and explore predictive questions about listing outcomes. Target definitions and modeling methods will be finalized as the analysis develops.

> **Project status:** Work in progress. No EDA results or model performance are reported until computed from the source files.

## Workstreams

| Member | Area |
| --- | --- |
| Dzmitry Prybysh | Data understanding, integration, and exploratory data analysis (EDA) |
| Marvin Pearson | Prediction setup and baseline modeling (proposed) |
| Rett Kinsey | Review text, sentiment, and topic analysis (proposed) |

## Project layout

```text
seattle-airbnb-ml/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── README.md
├── notebooks/
│   └── 01_data_integration_eda.ipynb
├── src/
│   └── README.md
└── outputs/
    └── figures/
        └── README.md
```

## Source data

The five input datasets are currently in the [team Google Drive project folder](https://drive.google.com/drive/folders/10ERrXETxYL9nknCJd9Ar7y88v1bNHgeq):

| File | Typical data grain |
| --- | --- |
| `listings.csv` | One row per listing |
| `calendar.csv` | One row per listing and day |
| `reviews.csv` | One row per review |
| `seattle_holidays_events_2016.csv` | One row per holiday/event |
| `seattle_walkscore_raw.csv` | One row per neighborhood |

The raw CSV files are **not yet committed**. To run the notebook, copy the original CSVs into the local `data/` directory, keeping their filenames unchanged. See [data/README.md](data/README.md) for details.

## Setup

Python 3.10+ recommended:

```bash
git clone https://github.com/duskat/seattle-airbnb-ml.git
cd seattle-airbnb-ml
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Open [`notebooks/01_data_integration_eda.ipynb`](notebooks/01_data_integration_eda.ipynb). The notebook begins with data inventory and profiling `listings.csv`; subsequent analysis will be added incrementally.

## Analysis guidelines

- **Respect data grain:** `listings`, `calendar`, and `reviews` are not one-to-one. Aggregate appropriately before merging to avoid duplicate multiplication.
- **Avoid data leakage:** Define the prediction target, observation cutoff, and split before fitting transforms or building future-looking features.
- **Calendar availability is not observed occupancy:** `available=False` may represent host-blocked dates, not guest bookings. Estimated revenue or occupancy derived from calendar fields must be labeled as proxies.
- **Validate neighborhood names:** Walk Score coverage does not necessarily match the listings' neighborhood names exactly.
- **Document assumptions:** Save notebook outputs and report limitations with each substantive finding.

## Near-term schedule

- **Tuesday, October 20, 2026:** Continue individual work; team call to review, resolve outstanding issues, and finalize the progress report.
- **Wednesday, October 21, 2026:** Progress Report submission.
