# unwrap-data

Raw datasets used by the `jgraving/unwrap` analysis notebooks. Kept in a
separate repository so that `unwrap` itself stays small — clone or download
only the parts you actually need.

The two locust experiments were conducted at different times by different
people; their data live in separate directories and are unrelated.

## `directional/` — locust directional-preference data

Per-individual raw tracking data from the Sayin et al. (2025) VR directional
experiment. Each file is one locust's full trajectory.

- Split **losslessly** from the original `XY_raw_50cm_.h5` (32 animals,
  `XY_raw_50cm__*.parquet`) and `XY_raw_8cm_.h5` (22 animals,
  `XY_raw_8cm__*.parquet`) tables. The split files reassemble to exactly
  the original 6,247,337 rows.
- Columns: `X` (float64), `Y` (float64), `fname` (str, the animal/trip id
  also encoded in the filename).
- The original `.h5` files are >100 MB each (git limit); these per-animal
  parquets are the same data in a form that git can carry.

Downstream preprocessing (rediscretization to fixed step length, direction
annotation) is in the `unwrap` repo and in
`jgraving/sayin_locust_mixture_model`.

## `density_order/` — locust density/order data

Raw tracking data from the density-order experiment, as parquets switched
from the original tracking-export format (no further raw form exists).
Files are size-capped parts, not logical units — read them all:

```python
import glob, pandas as pd
files = sorted(glob.glob("density_order/*.parquet"))
df = pd.concat([pd.read_parquet(f) for f in files], ignore_index=True)
```

- Columns: `X` (float64), `Y` (float64), `fname` (str), `loss` (float),
  `order` (float; local orientation order parameter), `density` (float;
  local neighbour density), `groups` (int; trial grouping).
- 107 files, 53,226,544 rows in total.

## `beetle_security_light/` — beetle security-light preference data

Beetles' heading in a light-on vs. light-off apparatus, from the 2019
security-light experiment. Single tab-separated file (one row per beetle,
trial, and stimulus presentation):

- `beetle_security_light/SecurityLightBergsig.txt`
  (`Experiment`, `Beetle`, `Trial`, `Heading` in degrees).

The `unwrap/beetles.ipynb` notebook loads it, converts headings to radians,
and refits a circular mixture model of light preference per beetle.

## `zebra_bearings/` — zebra escape-bearings data

Pre-approach vs post-approach predator-escape bearings (the input to
`unwrap/zebra_escape.ipynb`):

- `zebra_bearings/zebra_bearings_blocks.parquet` — preprocessed block bearings
  (300-frame mean direction per track and condition):
  `Track` str, `Observation` str, `Condition` ordered str
  {`pre-approach`, `post-approach`}, `HeadingRadians` float64,
  `PostStimulus` int. 5,417 rows (observation088, all tracks with both
  conditions).
  This is an *exported view* of the raw data below — it can be reproduced
  exactly from `zebra_bearings/raw/`.
- `zebra_bearings/raw/*.parquet` — the raw per-frame bearings, one file per
  tracked animal (59 files, 4 zebras, 3,897,536 frames). Columns:
  `observation` (str), `track_number` (str), `frame` (int),
  `compass_bearing` (float, degrees), `pre_scare` (bool), `row_index` (int,
  the row number in the original CSV; needed to reassemble the exact
  original ordering). Split losslessly from `zebra_bearings.csv`; the files
  reassemble to exactly the original 3,897,536 rows.

## Notes

- All files are read-only research data; do not modify in place.
- Originals for the directional experiment are also released at
  [zenodo.org/doi/10.5281/zenodo.14353283](https://doi.org/10.5281/zenodo.14353283).
