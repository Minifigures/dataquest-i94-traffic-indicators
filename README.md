# Heavy Traffic on I-94: Checking the Standard Answer

Dataquest guided project, *Finding Heavy Traffic Indicators on I-94*. Dataset: Metro Interstate Traffic Volume (John Hogue, UCI Machine Learning Repository), hourly westbound traffic between Minneapolis and St Paul with weather, 2012-10-02 to 2018-09-30.

## The claim I checked

This dataset comes with a well-known set of conclusions. The weather half says heavy traffic is indicated by **shower snow**, **light rain and snow**, and **proximity thunderstorm with drizzle**.

Snow predicting *heavy* traffic is an odd result, so I looked at how many observations it rests on. The answer is **1, 4 and 6 rows**.

Rebuilding the analysis reverses the finding, and turns up two data problems that explain where it came from.

## Findings

### 1. The standard answer has no sample behind it

| Condition | Hours | Effect vs time baseline | 95% CI |
|---|---|---|---|
| shower snow | 1 | not computable | n/a |
| light rain and snow | 6 | +19 | [-98, +137] |
| proximity thunderstorm with drizzle | 13 | -51 | [-215, +114] |

Both computable intervals straddle zero widely. This is the top of an unfiltered ranking, not a result.

### 2. The same data says the opposite, on real samples

| Condition | Hours | Effect | 95% CI |
|---|---|---|---|
| snow | 293 | **-553** | [-650, -457] |
| light snow | 1,938 | **-253** | [-282, -223] |
| heavy snow | 616 | **-250** | [-312, -187] |
| Snow (all) | 2,795 | **-285** | [-311, -258] |

Snow is the largest single weather effect in the dataset, and it is strongly negative. Reading the full table: five conditions clearly reduce traffic (Snow -285, Fog -149, Haze -122, Mist -82, Thunderstorm -39), only Clear skies clearly increase it (+44), and Rain, Clouds, Drizzle, Smoke and Squall are indistinguishable from a typical hour. **The best weather adds 44 vehicles an hour; snow removes 285.**

### 3. The table is not one row per hour

It is one row per hour-and-weather-condition pair. 5,386 timestamps carry two or more weather descriptions, with the single traffic reading copied into every row. **7,629 of 48,204 rows (15.8%) repeat an hour already counted**, and one hour appears up to six times. No timestamp ever carries two different traffic readings.

### 4. That duplication is confounded with the variable being studied

Share of each weather type's rows that share their hour with another row:

| Weather | Duplicated |
|---|---|
| Fog | 95.0% |
| Drizzle | 91.6% |
| Thunderstorm | 76.9% |
| Mist | 71.8% |
| Snow | 57.2% |
| Rain | 53.6% |
| Clouds | 0.8% |
| Clear | 0.5% |

Conditions that rarely occur alone get written repeatedly; conditions that always occur alone do not. A row-level comparison across weather weights the bad-weather sample two to six times more heavily than the fair-weather one. The aggregate daytime mean only moves 33 vehicles on deduplication because the bias cancels in a mixed sample, but it does not cancel inside a single weather category, which is exactly where the question is asked.

### 5. Weather was never the main story

| Factor | Variance explained |
|---|---|
| hour of day | **78.2%** |
| hour x day of week | **93.6%** |
| day of week | 5.1% |
| weather | 2.1% |
| month | 0.4% |

A business-day peak hour carries about **17 times** the 2am trough (6,241 against 373). Business days run 36% above weekends. The seasonal spread of 390 vehicles between August and December is barely larger than the snow effect and a fraction of the 5,335 swing within one day.

### 6. Other data quality notes

- **10 rows record a temperature of 0 Kelvin**, absolute zero. Sensor dropouts coded as zero rather than null.
- **One row records 9,831.3 mm of rain in an hour.** The world record is roughly 305 mm.
- **17 rows are fully identical** across every column.
- **22.8% of calendar hours are missing** (40,575 observed against 52,551 in the period).

## Method

1. Collapse to one row per hour.
2. Build a time baseline: median traffic for each of the 168 day-of-week by hour cells.
3. Take each hour's residual against its own cell, re-centred on the overall mean residual.
4. Group residuals by weather label, treating weather as multi-label so an hour that was both rainy and misty contributes to both.
5. Report 95% confidence intervals, so sample size is visible rather than hidden.

## What I would not claim

- **One direction on one road.** Nothing here generalises to other corridors or to eastbound traffic.
- **The baseline is a median, not a model.** It absorbs the weekly cycle but not holidays, roadworks, or six years of traffic growth.
- **Effects are associations.** Snow hours are also winter hours, dark hours, and school-closure hours. The time baseline removes the weekly cycle, not the seasonal and daylight confounds.
- **The intervals assume independent hours, which they are not.** Traffic is autocorrelated and weather arrives in multi-hour spells, so true intervals are wider than printed. The snow result survives that comfortably; marginal ones like Thunderstorm at -39 may not.

## Files

- `notebook.ipynb`: full analysis, executed, with three charts.

## Stack

pandas, NumPy, Matplotlib (fivethirtyeight style). No seaborn.