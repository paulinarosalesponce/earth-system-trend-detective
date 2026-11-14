# Use of Super Intelligence (SI)

NASA Space Apps requires teams to state clearly where and how SI tools were used.
This is a complete account for this project.

> **Please review and edit this file before submitting** so that it describes your own
> process accurately. It is a draft based on how the work actually ran, but you are the
> only person who can confirm it.

## Tool used

**Claude (Anthropic)**, used through the Claude desktop application, as a coding and
research assistant throughout the hackathon.

## What the team member decided

The direction of the project was set by the team, not by the SI tool:

- Selecting the challenge **"Be An Earth System Trend Detective!"**
- Deciding the solution should be a **web page** rather than a notebook or desktop app
- Deciding the tool must connect to **live NASA services** rather than ship with
  pre-downloaded data
- Deciding the interface should centre on a **world map**, with the Esri article
  *"Six ways to visualize change over time in web maps"* as the visual reference
- Approving the scope: one person, a day and a half, linear trends only
- Final review and submission

The structure of the analysis itself — select a variable, acquire it for a region,
plot a time series, test the trend for significance — is specified by the NASA
challenge text and was followed as written.

## What the SI tool did

- Researched which NASA data services can be called from a browser without a login,
  and confirmed by direct testing that the NASA POWER API allows cross-origin requests
  and that its regional endpoint is limited to 10° × 10° per request
- Wrote all of the HTML, CSS and JavaScript in `index.html`
- Implemented the statistics: ordinary least squares with a lag-1 autocorrelation
  correction, the Mann–Kendall test with tie correction, and the Theil–Sen slope with a
  distribution-free confidence interval
- Verified that implementation against SciPy and pymannkendall on six synthetic test
  series, and corrected two defects found that way (an incorrect Sen's slope confidence
  interval, and a missing-year bug in the demo presets)
- Ran every worked example in the tool against live NASA POWER data to confirm the
  results quoted are real
- Wrote the variable definitions, the explanatory text and this documentation

## Generated media

No SI-generated images, video or audio appear in this project. The interface is
hand-written code; the world map is drawn from Natural Earth coastline data; every
number shown in the tool is computed from NASA POWER data at the moment the user
requests it.

## NASA branding

No NASA logos, flags, mission identifiers or other NASA design elements are used
anywhere in the project. NASA is credited in text as the source of the data.
