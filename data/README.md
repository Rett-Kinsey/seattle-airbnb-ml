# Data inputs

Place the team's five original CSV files in this directory when working locally:

| File | Grain | Keys / fields to verify |
| --- | --- | --- |
| `listings.csv` | Listing | `id`, `neighbourhood_cleansed` |
| `calendar.csv` | Listing-day | `listing_id`, `date` |
| `reviews.csv` | Review | `listing_id`, `date` |
| `seattle_holidays_events_2016.csv` | Holiday/event | `date`, `neighborhood_focus` |
| `seattle_walkscore_raw.csv` | Neighborhood | Name, walk/transit/bike scores |

The originals live in the [project Google Drive folder](https://drive.google.com/drive/folders/10ERrXETxYL9nknCJd9Ar7y88v1bNHgeq).

## Handling notes

- Profile each table before joining. Listing, listing-day, review, and neighborhood records have different grains.
- **Do not** join calendar and reviews directly on `listing_id`; that would multiply records. Aggregate reviews to the appropriate time/listing grain first.
- Verify holiday-event date coverage and neighborhood matching rather than assuming every source covers identical dates or areas.
- Availability is not proof of booking. Any estimated occupancy/revenue must be described as a proxy.
- Avoid committing credentials, raw personal contact data, or copyrighted course materials to this public repository.
