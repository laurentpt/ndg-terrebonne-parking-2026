# Inputs

Open-data snapshots read by the study configuration (`config/`). Paths in the configs point
here.

## OpenStreetMap (`osm/`)

Overpass API extracts, © OpenStreetMap contributors, [ODbL 1.0](https://opendatacommons.org/licenses/odbl/1-0/).
Street centrelines and cross streets: the street axis, block boundaries and the block faces
of the area capacity.

| file | OSM data as of | query bbox (lat, lon) | sha256 |
|---|---|---|---|
| `terrebonne_full_2026-10-23.json` | 2026-10-23 21:48 UTC | 45.45687, -73.64382 – 45.48201, -73.62107 | `994eaeb232bfdd85d6ac9cd095659ba8d43425d03334ea89eef01f60db7cd5a4` |

## City of Montreal (`montreal_2026-10-23/`)

Ville de Montréal open data, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/),
downloaded 2026-10-23. The study area is a small part of each city-wide table: these files
keep the rows inside the OSM query bbox above, padded by 0.002° (about 200 m), every column
unchanged. The study's figures are identical with the subsets and the full tables (checked
2026-11-17).

| file | dataset | rows (subset / city) | sha256 of the city-wide download | sha256 of the subset |
|---|---|---|---|---|
| `signalisation_stationnement.csv` | [Signalisation de stationnement sur rue](https://donnees.montreal.ca/dataset/8ac6dd33-b0d3-4eab-a334-5a6283eb7940) | 3663 / 144805 | `f19b30f3704323bce951b101ec1fe02628424aa4a3526d28af4dd11c4435f04a` | `d952e8a8ee665e897e978d41682ba58917a4d3eadc31125bfe6de75e09c60d45` |
| `bornes_incendie.csv` | [Bornes d'incendie](https://donnees.montreal.ca/dataset/cb4de65e-138b-4936-9d5c-2d9a0bc9b4ce) | 505 / 31370 | `3447c1436ed6ad52f4846c876fb6ad4c3948e47d6db6766c1a7c86060abc1f48` | `b4ae1bdee3400b8f5d56246611a1a5726484c94fb7a872913e9a370ba4c173fa` |

## Labels and corrections made for the study

These are in `config/streets/`, next to the street they apply to:

- driveways (curb cuts) drawn on the curb images and reviewed one by one: `*_curb.yaml`;
- sign corrections against the city data (dropped poles, mirrored arrows, moved poles) and
  site notes: in each street config, `regulations.sources`.
