# Validation

What supports the counts and the curb model behind the NDG 2026 occupancy figures. 
Most of the checks below compare the pipeline with itself: two detectors,
two passes over the same cars, night against dusk, signs against where cars actually park.
Section 1 compares it with a hand count. No per-car data or images are published (privacy),
and the analysis scripts behind these checks are not part of the release.

What is counted: every vehicle parked in the parking lane of the recorded side, per block
(one side of the street between two cross streets) and per pass. That includes vehicles in
non-parkable spots (driveways, corners). A vehicle whose centre lies on a cross street's
roadway is not parked (turning or crossing) and never counts. Occupancy = those vehicles /
the block's capacity under the regulations in force at the time of the pass (see the map's
method).

## 1. Hand count

Two Terrebonne passes counted by hand from the raw video, block by block, without seeing the
pipeline's numbers: one dusk pass (GX014486, 2026-10-26 17:38, shutter 1/960) and one night
pass (GX014504, 2026-11-10 20:17, shutter 1/60). Per-block sheets: `handcount/`.

| pass | hand | pipeline | difference | blocks equal | blocks within 1 |
|---|---|---|---|---|---|
| dusk, 10-26 | 85 | 84 | -1 | 23 / 26 | 26 / 26 |
| night, 11-10 | 89 | 88 | -1 | 24 / 26 | 25 / 26 |
| both | 174 | 172 | -2 (-1.1 %) | 47 / 52 | 51 / 52 |

The totals agree within one vehicle per pass. Each pass has one real miss, found on the
mosaics and the frames. At dusk it is a car partly hidden by passing traffic
(Bessborough-Benny). At night it is a dark, unlit vehicle right next to the camera, which the
detector scored 0.16-0.30 (Hingston-Hampton). The other block differences are a vehicle near
a block end counted in the neighbouring block, so they cancel in the totals.

Counting by hand at block level is not error-free either. The first hand count put cars near
the Prince of Wales and Cumberland crossings in the wrong block. Cumberland meets the street
on the other side only, so the counted curb has no visible block end there.

## 2. Detection

**Two detectors.** The released detector (D-FINE-X, Apache-2.0) was compared with a second
one (YOLO11-L) on all 32 study mosaics of 2026-10-23 to 10-29. Both go through the
same split-car merging and geometric acceptance; boxes were matched by along-street overlap
(IoU >= 0.5). 1253 of 1258 YOLO vehicles were matched, with 5 YOLO-only and 9 D-FINE-only.
The result is flat over confidence thresholds 0.3-0.5. Of the 7 disagreements on counted
rows, checked by eye: D-FINE was right on 2 (a dark SUV, a truck YOLO split in two), YOLO on 2
(an occluded SUV, a moving car), and 3 were unclear (trailers, one dark vehicle). Swapping
detectors moved the area's evening occupancy by under one point.

**Night.** The late slot (19-21 h) was recorded in full dark at shutter 1/60, ISO 6400, after
a trial night comparing 1/120 and 1/60. Detector confidence of kept vehicles:

| passes | conf. median / 10th pct. | kept boxes < 0.5 |
|---|---|---|
| dusk, 1/960 (Terrebonne) | 0.95-0.96 / 0.91-0.94 | 0 |
| night trial, 1/120 (Terrebonne) | 0.80-0.83 / 0.43-0.52 | 3-7 per row (not used) |
| night, 1/60 (Terrebonne, 4 nights) | 0.93-0.94 / 0.86-0.90 | 0-3 per pass (dropped) |
| night, 1/60 (side streets) | 0.88-0.93 / 0.52-0.89 | 0-2 per street (dropped) |

At 1/120, the near-row cars were unlit silhouettes. Borderline boxes there were about half
false positives and half missed cars, which cancelled in the totals, so 1/120 was dropped.
At 1/60, every low-score box checked was clutter or a fragment of a kept car, so night
passes use a 0.5 confidence threshold.

## 3. Repeat passes over the same cars

Out and back on the same evening, a few minutes apart (no labels involved):

| pair | vehicles | matched | mean \|block difference\| | blocks within 1 car |
|---|---|---|---|---|
| dusk, 10-23 (8 min apart) | 96 / 98 | 93 | 0.33 | 24 / 27 |
| night trial, 11-05 | 57 / 54 | 40 | 0.31 | 15 / 16 |

Block counts agree as well at night as at dusk. Matching individual cars is worse at night,
mostly because of placement: the cars counted in only one pass were mostly found in both
images, one car apart.

**Night against dusk, same block faces.** Where a side street was ridden at dusk and at night,
the night counts fall within or just above the range of the dusk evenings. On Beaconsfield
(Somerled-Terrebonne) the three nights counted 17 / 16 / 18 against 20 / 12 / 16 at dusk. On
Royal (Terrebonne-Somerled) they counted 21 / 17 / 20 against 18 / 17 / 19. On Wilson
(Terrebonne-Monkland) they counted 26 / 22 / 25 against 24 on the one dusk pass that covered
that face. So there is no sign of a night undercount. Very low night counts were checked on brightened
frames: King Edward south of Terrebonne (2 cars over 259 m) is an empty curb, and the
vehicles there were in driveways.

## 4. Placement along the street

Each pass is placed on a common street axis (an OpenStreetMap centreline), so that its cars
land in the right block and against the right driveways and signs. GPS alone is 3-15 m off
along the street at dusk. Matching parked-car patterns locks one or two cars off in places,
because parked cars are roughly periodic. Passes are therefore placed by registering the
background of the curb images (house fronts, trees, poles) onto other passes riding the same
way. That does not depend on the cars.

Checks on the 16 non-reference evening passes (756 counted vehicles), before and after
registration:

| | before | after |
|---|---|---|
| vehicles on driveways that were labelled on another pass | 55 (7.3 %) | 20 (2.6 %) |
| floor: on the passes the driveways were drawn on | 6 / 369 (1.6 %) | |
| pass-to-pass residual, independent pairs (median) | 0.6-13 m | 0.04-0.3 m |
| pass-to-pass residual (90th percentile) | 2.5-19 m | 0.2-1.2 m |

Night passes cannot register onto daylight images (the background is unlit), but they
register onto each other: on every street, each later night registers onto the first one
(Terrebonne residual 0.03 m; side streets <= 0.04 m). The first night of each street is placed
by GPS (DOP ~2 at night) plus parked-car patterns limited to +-5 m. On the trial night, that
put cars 0.6-0.9 m (median) from the matching dusk cars.

## 5. Regulations against parked cars

Capacity comes from the City of Montreal signage open data. On rue de Terrebonne, every panel
was first applied as recorded (Quebec arrow convention). That put 261 of 927 parked-vehicle
observations inside no-stopping zones, with a street capacity of 102 spaces. Most of the
conflict came from whole block faces whose corner panels were recorded with their arrows
mirrored (all installed the same week in May 2024). The smallest set of record corrections
that removes the conflict is 11 arrows and 2 panels. After those, 28 observations (3 %) were
inside no-stopping zones, all at corner zones where a car overlaps the pole end on some
evenings. Pole positions were then checked on the images (9 poles placed where they stand, and the
Old Orchard corner panel dropped: it stands at the corner, inside the intersection clearance),
and two cross-street edges were corrected by hand. On the final model, 3.9 vehicles per
Terrebonne pass are not in a legal spot. They still count in the numerator. Every correction is listed with the study data.

On the side streets, each zone's direction is inferred from the panels of its block face. Those
streets have not had a check like Terrebonne's.

Driveways are deducted from capacity. On rue de Terrebonne they were drawn by hand on the curb
images (46). On the side streets an image classifier trained on those labels (not released)
proposed candidates on the curb images, and each was reviewed by the author: kept, redrawn or
rejected. Of the 204 side-street labels, 203 were confirmed; one, marked uncertain, was kept as
proposed (Wilson, Somerled-Terrebonne). Labels drawn on a single daylight pass (2026-11-12 and 11-15) were moved
to the night passes' placement using the parked cars those passes share.

## 6. Occlusion by passing traffic

A vehicle between the camera and the parked row hides the curb. Such events are found from
image motion: a stopped or oncoming vehicle in the travel lane is nearer than the parked row,
and an overtaking one moves backwards in the image. Over the dusk passes, 0-3.6 % of the curb
was occluded per pass (186 m in 60 events). The worst case was Terrebonne's far row, with
overtaking SUVs and minivans. Of the 8 longest events checked, 7 were real occluders. The
vehicles possibly missed in those events number about 7 of 1125 counted (0.6 %), below the
variation from one evening to the next. No correction is applied. Blind spot: a vehicle
riding alongside at the bike's speed reads as a gap and is not caught.

## 7. Known limits

- **Timing.** Weekday evenings only. Early slot (17-19 h): passes started 17:17-18:01 (dusk,
  5 evenings on Terrebonne). Late slot (19-21 h): passes started 19:57-20:46 (dark, 4 nights).
  So each slot's figure rests on about one hour of it. No weekends, no overnight count.
- **Season.** Late October to mid-November, no snow; dry except the 2026-11-05 night (wet
  road). Snowbanks reduce winter capacity.
- **Lighting.** The 10-29 dusk passes were shot at 1/240 (darker, more blur). Speed came from
  GPS on several passes because image motion fails in low light.
- **One side per side street.** Side streets were measured on the side reachable riding with
  traffic. The capacity view extrapolates measured spaces per metre to the faces that were
  not ridden; the sample was chosen, not drawn at random.
- **Streets in one slot only.** Eight of the twelve streets were ridden in one slot (dusk or
  night). That slot stands for both; the map flags them.
- **Permit zones** (resident permits, S3R) count as capacity.
