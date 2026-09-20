# Pakistan Mutual Funds — Daily NAV Dataset

Daily net asset values, dividend payouts and total returns for ~550 Pakistani
mutual funds, held against the KSE-100 and KMI-30, updated automatically every
business day from [MUFAP's](https://www.mufap.com.pk/) public pages and the
[PSX Data Portal](https://dps.psx.com.pk/). No signup, no API key — just clone
or fetch the raw files.

Built by the scraper behind
[pakistan-mutual-funds-api](https://github.com/saadsalmankhan/pakistan-mutual-funds-api);
run that if you want a live REST API with filters and history endpoints
instead of flat files.

## Files

| Path | Contents |
|---|---|
| `funds.json` | Latest snapshot: every fund's current NAV, offer price, AMC, category, Shariah flag, benchmark (KSE-100 / KMI-30 / null), expense ratio, management fee and inception date, plus `updatedAt` |
| `history/<fundId>.ndjson` | One line per business day per fund: `{"date","nav","offerPrice"}` — accumulates daily |
| `meta.json` | Per-fund metadata from MUFAP's Expense Ratios table: TER YTD %, management fee %, inception date |
| `payouts/<fundId>.ndjson` | Every dividend/payout since 2022: `{"date","payout","exNav"}`, PKR per unit and the NAV right after it. No file means the fund never paid out |
| `indices/KSE100.ndjson`, `indices/KMI30.ndjson` | End-of-day index closes: `{"date","close"}`. Both are total-return indices |
| `performance.json` | Who beat the market after fees: every fund's total returns (1m to 3y, payouts reinvested) next to its benchmark index over the same dates |

`fundId` is MUFAP's internal fund id (the `FundID` in their fund-detail URLs).

## Who beat the market

`history/` alone will mislead you about any fund that pays dividends: a
payout drops the NAV by the amount paid without the investor losing a rupee.
Faysal Islamic Stock Fund's NAV rose 16% over the three years to 18 Sep 2026.
With its two payouts reinvested the return was 190%, which is also what MUFAP
reports. `performance.json` does that arithmetic for every fund and sets the
result beside the index:

```bash
curl -s https://raw.githubusercontent.com/saadsalmankhan/pakistan-mutual-funds-data/main/performance.json \
  | jq '[.funds[] | select(.category=="Equity" and .stale==false and .returns["3y"].anomalies==0)
         | {name, pct: .returns["3y"].pct, index: .returns["3y"].benchmarkPct, gap: .returns["3y"].excessPct}]
        | sort_by(-.gap)'
```

Per fund and period: `pct` (total return, net of fees, cumulative not
annualized), `navPct` (NAV-only), `payouts` (how many were reinvested),
`benchmarkPct` (KSE-100 for conventional equity categories, KMI-30 for Shariah
ones, `null` elsewhere), `excessPct` (fund minus index, percentage points) and
`anomalies`. Skip a return whose `anomalies` is above 0: the window holds a
NAV level shift no payout explains, usually a unit consolidation or an error
in the source. `passive` marks index trackers and ETFs, `stale` marks funds
that stopped reporting.

The figures match MUFAP's own payout-adjusted returns within 1 percentage
point at 1 and 2 years for every equity fund checked (Sep 2026). Method,
validation and the caveats that belong next to any league table (closed funds
are missing, so the industry looks better than it was) are in the
[API README](https://github.com/saadsalmankhan/pakistan-mutual-funds-api#total-returns-and-beating-the-market).

## Fetch examples

Latest snapshot:

```bash
curl -s https://raw.githubusercontent.com/saadsalmankhan/pakistan-mutual-funds-data/main/funds.json
```

One fund's NAV history (ABL Cash Fund):

```bash
curl -s https://raw.githubusercontent.com/saadsalmankhan/pakistan-mutual-funds-data/main/history/12768.ndjson
```

## Update cadence & data notes

- A GitHub Action runs every business day (scheduled 21:30 PKT, with a
  second slot at 23:30 PKT): it refreshes the snapshot from the Fund
  Directory, merges the trailing ten days of NAV history and 45 days of
  payouts from MUFAP's daily-stats tables, merges index closes from PSX and
  rebuilds `performance.json`. If Cloudflare blocks the runner it re-dispatches itself
  onto a fresh one (up to three attempts), and whatever a partial run did
  fetch is committed anyway. The commit log is the audit trail.
- History dates are MUFAP's published NAV validity dates, at MUFAP's full
  4-decimal precision — however late the workflow actually runs, rows land
  under the right day, and MUFAP's own corrections get picked up by the
  trailing-window re-merge.
- History is backfilled from 2022-01-01 onward and accumulates daily.

## Disclaimer

Data is scraped from MUFAP's public Fund Directory and provided as-is, for
informational use only. This project is not affiliated with or endorsed by
MUFAP. Verify against MUFAP directly before making any financial decision
based on this data.

## Author

Built by [Saad Salman](https://saadsalman.org).
