# Earth System Trend Detective

**NASA Space Apps Challenge 2026 — "Be An Earth System Trend Detective!"**

A single-page web tool that lets anyone pick a NASA Earth science variable, pick a
place on the map, and get a statistically defensible answer to four questions:

> **What** is changing · **where** is it changing · **how much** is it changing ·
> and is the change **significant**?

**▶ Live tool:** _(add your GitHub Pages URL here)_

---

## What it does

1. **Choose a variable.** 25 Earth science variables grouped into four themes
   (Heat, Water, Air, Sun & sky), each with a plain-language definition written for a
   high-school or general-science reader, plus a note on *why it might trend*.
2. **Choose a place.** Click a point on the world map, or drag a box of up to
   10° × 10° to analyse many grid cells at once.
3. **Read the verdict.** The tool fetches the monthly record live from NASA POWER,
   removes the seasonal cycle, fits a trend, runs three independent significance
   tests, and reports the result in plain English alongside the full statistics.

When a box is analysed, every NASA grid cell inside it is tested separately and drawn
as a coloured dot, with a white ring where that cell's trend is significant. This makes
the central difficulty of the challenge visible: **the same variable can trend in
opposite directions in neighbouring parts of one region.**

## Data

All data comes live from the **[NASA POWER API](https://power.larc.nasa.gov/)**
(Prediction Of Worldwide Energy Resources) — no API key, no login, no cached copies.

- Monthly time series, **1981 to present** (three radiation variables start in 1984;
  aerosol optical depth and UV-B start in 2001 — the tool says so when it happens).
- POWER's meteorological variables are derived from NASA's **MERRA-2** reanalysis;
  its radiation variables come from NASA GEWEX/CERES.
- Native grid: **0.5° latitude × 0.625° longitude**.
- Variable names, units and definitions follow the POWER parameter catalogue.

Endpoints used:

| Purpose | Endpoint |
|---|---|
| Single point | `/api/temporal/monthly/point` |
| Grid box (≤ 10° × 10°) | `/api/temporal/monthly/regional` |

## The statistics

Monthly values are parsed (POWER's month-13 annual summary and its `-999` fill values
are discarded), then the **seasonal cycle is removed** by subtracting the mean for each
calendar month. Seasonal swings are usually far larger than the trend and hide it.

Three independent tests then run on the anomaly series:

| Test | What it assumes | What it gives |
|---|---|---|
| **Ordinary least squares + t-test** | a straight line, roughly normal residuals | rate of change, 95% confidence interval, R² |
| **Mann–Kendall** | nothing about shape or distribution | S, Z, Kendall's τ, p-value |
| **Theil–Sen (Sen's slope)** | nothing; robust to outliers | median pairwise slope + distribution-free CI |

### Autocorrelation correction

Monthly climate anomalies are serially correlated — a warm month tends to follow a warm
month — so consecutive observations are *not* independent evidence. Treating all 540
months as independent makes almost any wiggle look significant.

The tool measures the lag-1 autocorrelation **r₁** of the regression residuals and
reduces the sample size to an effective

```
n_eff = n · (1 − r₁) / (1 + r₁)
```

The t-test uses `n_eff − 2` degrees of freedom. The tool shows r₁ and n_eff so you can
see exactly how much evidence was discounted, and warns when n_eff becomes too small to
trust. Mann–Kendall and Sen's slope are reported unadjusted.

### Verification

The statistics engine was checked against **SciPy** (`scipy.stats.linregress`,
`scipy.stats.t`) and **pymannkendall** on six synthetic series covering a clean trend,
pure noise, skewed and tied data, a short record, a strongly autocorrelated series and a
series with missing values. OLS slope, standard error, t, p, R², Mann–Kendall S, var(S),
Z, τ, p and the Theil–Sen slope all agree to within **1 × 10⁻⁹**.

## Running it

It is one self-contained HTML file with **no build step and no dependencies**.

```bash
# open it directly
open index.html

# or serve it
python3 -m http.server 8000
```

To publish: push to a GitHub repository and enable **Settings → Pages → Deploy from
branch → main / root**.

## Technical notes

- Vanilla HTML, CSS and JavaScript. No frameworks, no bundler, no CDN.
- The world map is drawn on `<canvas>` in an equirectangular projection. Coastlines,
  country borders and ~19,600 populated places are embedded in the file in a packed
  base-36 encoding, so the map works with no tile server and makes no request to any
  server other than NASA POWER. City names appear progressively as you zoom, and a
  search box finds any listed place by name (accents optional).
- Charts are generated as inline SVG.
- Per-cell analysis skips Theil–Sen (its O(n² log n) sort is too slow for hundreds of
  cells) and uses OLS + Mann–Kendall; a 10° × 10° box of 357 cells analyses in
  well under a second.

## Limitations

This tool fits **linear** trends. Real Earth system change can be non-linear, can shift
abruptly, and can be driven by cycles such as ENSO that are not removed here. A
significant linear trend is evidence of consistent directional change over the period
chosen — it is not a forecast, and it does not establish a cause.

Finding *no* significant trend is a real result. Some trends only emerge with a longer
record.

## Use of Super Intelligence (SI)

See `SI_DISCLOSURE.md`.

## References

- NASA POWER Project. https://power.larc.nasa.gov/ — data free to use with attribution (CC BY 4.0).
- Gelaro, R. et al. (2017), *The Modern-Era Retrospective Analysis for Research and Applications, Version 2 (MERRA-2)*, J. Climate, 30, 5419–5454.
- Mann, H.B. (1945), *Nonparametric tests against trend*, Econometrica, 13, 245–259.
- Kendall, M.G. (1975), *Rank Correlation Methods*, Griffin, London.
- Sen, P.K. (1968), *Estimates of the regression coefficient based on Kendall's tau*, JASA, 63, 1379–1389.
- Gilbert, R.O. (1987), *Statistical Methods for Environmental Pollution Monitoring*, Van Nostrand Reinhold. (Sen's slope confidence interval.)
- Santer, B.D. et al. (2000), *Statistical significance of trends and trend differences in layer-average atmospheric temperature time series*, JGR, 105, 7337–7356. (Effective sample size.)
- World coastlines and country borders derived from Natural Earth via the `world-atlas` package (public domain).
- Town and city names from [GeoNames](https://www.geonames.org/), licensed CC BY 4.0, via the `all-the-cities` package (MIT). Limited to places above ~20,000 inhabitants.
