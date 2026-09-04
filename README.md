# Pakistan Mutual Funds — Daily NAV Dataset

Daily net asset values for ~550 Pakistani mutual funds, updated automatically
every business day from [MUFAP's](https://www.mufap.com.pk/) public Fund
Directory. No signup, no API key — just clone or fetch the raw files.

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

`fundId` is MUFAP's internal fund id (the `FundID` in their fund-detail URLs).

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

- A GitHub Action runs once per business day (scheduled 21:30 PKT): it
  refreshes the snapshot from the Fund Directory and merges the trailing ten
  days of NAV history from MUFAP's daily-stats table. The commit log is the
  audit trail.
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
