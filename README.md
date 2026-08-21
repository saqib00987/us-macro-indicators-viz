# U.S. Macroeconomic Indicators: A Visual Analysis (FRED, 2000-2026)

A data visualization project on eight core U.S. macroeconomic indicators, pulled from the Federal Reserve Economic Data (FRED) API. The main questions are how changes in the Federal Funds Rate show up later in mortgage rates, markets, inflation, and unemployment, and whether consumer sentiment reacts more strongly to bad news than to good news of a similar size.

Written in Python with pandas, Matplotlib, and Plotly.

## Overview

The data covers eight FRED series from 2000 to 2026, resampled to monthly frequency (about 310 rows) and including the 2008 financial crisis, the 2020 COVID shock, and the 2022 inflation spike. The work is split into two notebooks:

1. `1_foundational_analysis.ipynb` sets up the basic picture with two reference charts: the Federal Funds Rate against unemployment, and the 10-year minus 2-year yield spread as a recession signal.
2. `2_transmission_and_sentiment.ipynb` does the deeper analysis: a lagged-correlation study of how rate changes spread through the economy, and a significance test of whether consumer sentiment reacts asymmetrically to good versus bad news.

## Dataset

The eight series are downloaded at run time from FRED's public CSV API (`https://fred.stlouisfed.org/graph/fredgraph.csv?id=<SERIES_ID>`) and resampled to month-end. No data files are stored in the repo, so a fresh run always uses the latest FRED numbers.

| Series | Description |
| --- | --- |
| `FEDFUNDS` | Effective Federal Funds Rate (%) |
| `DGS10` / `DGS2` | 10-year and 2-year Treasury yields (%) |
| `CPIAUCSL` | Consumer Price Index |
| `UNRATE` | Unemployment Rate (%) |
| `SP500` | S&P 500 Index |
| `MORTGAGE30US` | 30-year Fixed Mortgage Rate (%) |
| `UMCSENT` | U. Michigan Consumer Sentiment |

Derived columns include the yield-curve spread (`DGS10 - DGS2`), year-over-year inflation, and month-over-month changes used in the correlation analysis.

## The notebooks

### 1. Foundational views: [`notebooks/1_foundational_analysis.ipynb`](notebooks/1_foundational_analysis.ipynb)

Two charts to frame the data:

* Fed Funds Rate against unemployment, showing the delayed relationship between the policy rate and the labor market.
* The 10y-2y Treasury spread with recession shading, where the spread turns negative ahead of the NBER-dated recessions.

Fed Funds Rate against unemployment:

![Fed Funds Rate vs Unemployment](figures/1_foundational/a1_fedfunds_vs_unemployment.png)

Yield-curve inversion as a recession signal:

![Yield-curve spread with recession shading](figures/1_foundational/a2_yield_curve_spread.png)

### 2. Transmission and sentiment: [`notebooks/2_transmission_and_sentiment.ipynb`](notebooks/2_transmission_and_sentiment.ipynb)

The rate transmission part computes correlations on month-over-month changes rather than levels, so shared long-term trends don't produce false correlation. It measures the correlation between Fed Funds Rate changes and each variable at lags of 0, 6, 12, 18, 24, and 36 months, done separately for three periods (pre-2008, the zero-rate era of 2008-2020, and post-COVID). The final chart is a set of three side-by-side heatmaps so all three periods can be read at once. There is also an interactive Plotly version.

The sentiment part scores each month with a composite macro-stress measure, built by z-scoring the changes in unemployment and inflation so neither dominates because of its units. Months are grouped into quartiles by shock size, and within each quartile the median sentiment reaction to bad-news months is compared against good-news months of similar size. A Mann-Whitney U test checks whether each gap is real or just noise. The result is backed up two more ways: a swing-rate analysis that detects peaks and troughs in the sentiment series and compares the speed of drops against recoveries, and a test of the skew in the distribution of monthly changes.

One data note: FRED's `SP500` series only has real data from 2016 on, so the pre-2016 S&P 500 cells are marked as unavailable (hatched) instead of being filled with a constant that would give a false correlation.

Rate transmission across the three periods:

![Fed rate transmission heatmap](figures/2_transmission/b1_heatmap.png)

Sentiment reaction, matched by shock size, with significance testing:

![Sentiment asymmetry, magnitude-matched](figures/2_transmission/b2_composite_final.png)

## Findings

* Transmission to unemployment is faster after COVID. In the pre-2008 period, unemployment's correlation with earlier Fed Funds changes only turns positive around lag 36. Post-COVID the same turn happens at lag 6 (+0.44), about 30 months sooner.
* Inflation gives the clearest signal: positive at lag 0 post-COVID (+0.37) and turning negative by lags 18-24 (-0.32 to -0.38) as the rate hikes take effect.
* The S&P 500 correlations stay weak across all periods and get weaker post-COVID, which fits the idea that markets price in expectations rather than react to the realized change.
* Sentiment does react asymmetrically, but only for the largest shocks. The bad-versus-good gap is significant only in the top shock quartile (Q4, p = 0.045; median 3.40 vs 2.10 points). The smaller quartiles show no significant difference. Because Q4 would not survive a strict correction for running four tests, it's reported as suggestive rather than a firm result, and it lines up with the swing-rate analysis.

## Repository structure

```
us-macro-indicators-viz/
├── notebooks/
│   ├── 1_foundational_analysis.ipynb
│   └── 2_transmission_and_sentiment.ipynb
├── figures/
│   ├── 1_foundational/
│   └── 2_transmission/
├── environment.yml
├── requirements.txt
└── README.md
```

Both notebooks render fully in GitHub's preview, figures included.

## Running it

The notebooks download data from FRED when they run, so you need an internet connection.

With conda:

```sh
conda env create -f environment.yml
conda activate vis-project
jupyter lab
```

With pip and venv:

```sh
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Then open the notebooks in `notebooks/` and run all cells, starting with notebook 1.

## Notes

* The numbers reflect FRED data at run time and can move a little as series get revised.
* The Q4 sentiment result is marginal; the full significance discussion is in notebook 2.
* The S&P 500 series has no pre-2016 data on FRED, so those cells are left blank rather than filled in.

## Acknowledgment

Built for the Visualization for AI course (BSc Artificial Intelligence) at Johannes Kepler University Linz.

## License

MIT License. See [LICENSE](LICENSE).
