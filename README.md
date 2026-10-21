# Smart Mobility Le Mans

Automated dashboard of the SETRAM bus and tram network (Le Mans Métropole), built on
open GTFS / GTFS-RT data and GitHub Actions.

**Business question:** if Le Mans Métropole had limited resources to improve its public
transport network, where and when should it prioritize interventions?

## Pipelines

- **Static reference** (`build_static_reference.yml`, monthly): downloads SETRAM's GTFS
  feed, counts scheduled trips per stop on a representative weekday, and assigns each
  stop to its nearest commune.
- **Real-time snapshot** (`collect_snapshot.yml`, every 15 minutes): reads the GTFS-RT
  TripUpdate and Alert feeds, appends a per-route delay summary to the monthly history,
  and updates the live dashboard.

## Data sources

- SETRAM static GTFS and GTFS-RT feeds via transport.data.gouv.fr (open data, no key)
- INSEE legal population of Le Mans Métropole's 20 communes (demand proxy)

## Dashboard

Published with GitHub Pages from the `docs/` folder.
