# PourSignal — Data Dictionary & Methodology

PourSignal turns the Texas Comptroller's monthly Mixed Beverage Gross Receipts filings into
analysis-ready datasets: one clean record per venue per month, mapped to chains, parent companies
and stock tickers, with same-store growth already computed.

## Source and provenance

- **Source:** Texas Comptroller of Public Accounts, *Mixed Beverage Gross Receipts*
  (Texas Open Data Portal dataset `naix-2893`). Every holder of a Texas mixed-beverage permit
  files monthly, reporting liquor, wine, beer and cover-charge receipts per location.
- **Coverage:** ~22,500 reporting venues per month; history from January 2014 (2007 available on request).
- **Freshness:** filings are due on the 20th of the following month; the portal updates as returns
  are processed. PourSignal refreshes weekly and flags each month's *filing completeness*.
- **Nature of the data:** public government records. No consumer or cardholder data and no
  material non-public information. Owner names that appear to be private individuals
  (sole proprietors) are withheld in PourSignal products.
- **Receipts** are gross receipts as reported by the permittee and include the mixed beverage
  sales tax collected from customers. Growth rates are unaffected since the rate is constant.

## Key concepts

| Term | Definition |
|---|---|
| Venue / outlet (`unit_id`) | `taxpayer_number-location_number` — the Comptroller's identifier for one permitted location under one owner. A change of ownership starts a new `unit_id`. |
| Month | The month whose receipts are reported (obligation period end). |
| Comparable (comp) outlet | Reported positive receipts in month *t* and in *t-12*, and first reported at least 18 months before *t* (the 18-month rule most casual-dining chains use). |
| Comp growth | Σ receipts(t) ÷ Σ receipts(t-12) − 1, over comp outlets. Quarterly comp growth is re-derived from summed receipts, not averaged. |
| Filing completeness | Venues reporting in a month ÷ trailing 3-month average. Months below 95% are still filling in. |
| TTM | Trailing twelve complete months (ending at the latest month with completeness ≥ 95%). |

## Files

### `accounts_statewide.csv` — Account Intelligence (one row per active venue)

| Column | Description |
|---|---|
| `rank_statewide`, `rank_metro` | Rank by TTM total alcohol receipts |
| `unit_id` | Venue identifier (see above) |
| `location_name` | Trade name (DBA) as filed |
| `owner` | Permit-holding entity; individuals withheld |
| `chain_brand`, `chain_parent`, `chain_ticker` | Chain mapping where applicable |
| `address`, `city`, `zip`, `county_code`, `metro` | Venue location (metro = PourSignal MSA grouping) |
| `tabc_permit_number` | TABC permit number |
| `first_month`, `months_open` | First month this owner reported at this location |
| `opened_last_12m` | New owner/venue in the last 12 months |
| `ttm_total_receipts`, `ttm_liquor`, `ttm_wine`, `ttm_beer`, `ttm_cover_charge` | TTM receipts (USD) |
| `liquor_share` | Liquor ÷ total |
| `ttm_growth` | TTM vs prior TTM (venues open ≥ 24 months) |
| `last_month_receipts` | Receipts in the TTM end month |

### `brand_monthly.csv` / `ticker_monthly.csv` — Chain Tracker

| Column | Description |
|---|---|
| `brand`, `parent`, `ticker` | Chain mapping (ticker blank for private chains) |
| `month` | Reporting month |
| `outlets_reporting` | Venues with positive receipts |
| `receipts`, `liquor`, `wine`, `beer` | Total receipts (USD) |
| `receipts_per_outlet` | Average receipts per reporting venue |
| `comp_outlets` | Venues in the comp base |
| `comp_receipts`, `comp_receipts_ly` | Comp-base receipts this year / same month last year |
| `comp_growth` | Same-store alcohol sales growth |

### `brand_quarterly.csv` / `ticker_quarterly.csv`

Calendar quarters: `receipts`, `comp_growth`, `avg_comp_outlets`, `months_in_data`
(a quarter with fewer than 3 months, or whose last month is below 95% completeness, is quarter-to-date).

### `unit_brand.csv`

Every venue mapped to a chain, with first/last reporting month, for audit of the mapping.

## Chain mapping

Chains are matched by rules on the filing entity name and the venue trade name (e.g. filer
`CHILI'S BEVERAGE COMPANY, INC.` → Chili's → Brinker International → `EAT`). Rules are curated and
reviewed against filer/venue lists; the rule file ships with the Chain Tracker so the mapping is
fully auditable. Franchise-operated units are included under the brand (e.g. Applebee's → `DIN`).

## Caveats

- Alcohol is a fraction of total restaurant sales; alcohol comps track, but are not identical to,
  company-reported comps. See the published backtest.
- Texas is one state; read-through to a company total depends on its Texas footprint.
- Late or amended filings can revise recent months; the newest month is always provisional.
