# PPC Audit Compare Tool

A Nuxt 4 dashboard for daily cross-source data consistency checks across **GPP**, **SEC EDGAR**, and **fund websites**. For each fund (identified by CIK), it surfaces NAV discrepancies, distribution data, total net assets, shares outstanding, and GPP time-series charts — all in one place.

## Features

- **Comparison Dashboard** — filterable list of funds by date, fund name/CIK, and overall status (`match` / `mismatch` / `error` / `ok`)
- **Detailed comparison view** with six tabs (per fund):
  - **NAV Detail** — per-class NAV comparison across SEC, website, and GPP; website source URL shown inline
  - **Distributions** — gross distribution, servicing fee, and net distribution per class, plus fund-level data points (declaration/record/payment dates, total shares issued, SEC filing type, GPP data source, website URL)
  - **Total Net Assets** — per-class TNA comparison across GPP (ClassTNA section), SEC filing, and website; fund-level summary cards; match/mismatch status
  - **Shares Outstanding** — shares issued and percentage of total per class with visual allocation bar
  - **GPP Data** — time-series line charts (Price, Dividend, ClassTNA, CapitalGain, DailyDividend) with section availability badges
  - **Summary** — full side-by-side comparison matrix (GPP / SEC / Website) for all data points; website source URL linked in column headers
- **Ad-hoc file upload** — upload any `ComparisonDetail` JSON for instant preview; missing sections are filled with null defaults rather than rejected
- **Website source links** — wherever website data appears (NAV tab, TNA tab, Distributions tab, Summary tab), the website URL is shown as a clickable link
- **Typed data layer** — shared TypeScript interfaces for all comparison structures (`types/comparison.ts`)
- **Pluggable data adapter** — reads from local `data/` JSON files by default; set `DATA_SOURCE=s3` to switch to AWS S3 with no code changes

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Nuxt 4](https://nuxt.com) + [Vue 3](https://vuejs.org) |
| Styling | [Tailwind CSS](https://tailwindcss.com) |
| Charts | [Chart.js](https://www.chartjs.org) + [vue-chartjs](https://vue-chartjs.org) |
| Language | TypeScript |
| Runtime | Node.js (Nitro server) |
| Cloud storage | AWS S3 (optional, via `@aws-sdk/client-s3`) |

## Project Structure

```
app/
  pages/
    index.vue                        # Dashboard — date picker, search, status filter
    comparison/
      preview.vue                    # Ad-hoc preview for uploaded JSON files
      [date]/[cik].vue               # Fund detail page (6 tabs)
  components/
    tab/
      NavTab.vue                     # NAV per-class comparison + website URL
      DistributionsTab.vue           # Per-class distributions + fund-level data points
      TnaTab.vue                     # Per-class TNA comparison (GPP / SEC / Website)
      SharesTab.vue                  # Shares issued per class
      GppTab.vue                     # GPP time-series charts
    ComparisonMatrix.vue             # Summary: full side-by-side matrix with website links
    BaseTabs.vue                     # Reusable tab bar component
    StatusBadge.vue                  # match / mismatch / error / ok / unavailable badge
    CellValue.vue                    # Coloured value cell with optional sub-label
    DataRow.vue                      # Generic table row helper
    UploadModal.vue                  # JSON file upload with null-safe normalisation
  composables/
    useComparisons.ts                # Fetches CIK summary list for a given date
    useComparisonDetail.ts           # Fetches full ComparisonDetail for a CIK
    useUploadedComparison.ts         # Client-side state for ad-hoc uploaded file
server/
  api/
    dates.get.ts                     # GET /api/dates
    comparisons/[date].get.ts        # GET /api/comparisons/:date
    comparison/[date]/[cik].get.ts   # GET /api/comparison/:date/:cik
  utils/
    dataAdapter.ts                   # Local fs / S3 data layer (swap via DATA_SOURCE env var)
types/
  comparison.ts                      # Shared TypeScript interfaces
data/                                # Local audit data (git-ignored)
  <YYYY-MM-DD>/
    index.json                       # CIK summary list for the date
    <cik>.json                       # Full ComparisonDetail per fund
```

## Data Format

The app reads JSON files from `data/<YYYY-MM-DD>/`.

**`index.json`** — array of `CIKListEntry`:
```json
[
  {
    "cik": "1803498",
    "fund_name": "BlackRock Capital Allocation Term Trust",
    "overall_status": "mismatch",
    "timestamp": "2026-02-20T13:45:30Z"
  }
]
```

**`<cik>.json`** — full `ComparisonDetail` object (see `types/comparison.ts` for the schema). All top-level sections (`fund`, `sec_filing`, `comparison`, `distributions`, `shares_outstanding`, `performance`, `gpp_data`, etc.) are optional when uploading via the UI — any missing sections are normalised to null/empty defaults.

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `DATA_SOURCE` | `local` | Set to `s3` to read from AWS S3 |
| `S3_BUCKET` | _(none)_ | S3 bucket name (required when `DATA_SOURCE=s3`) |
| `S3_REGION` | `us-east-1` | AWS region for the S3 bucket |
| `S3_PREFIX` | _(none)_ | Optional key prefix inside the bucket, e.g. `data/` |

When `DATA_SOURCE=s3`, the app uses the IAM instance role for credentials — no access key / secret env vars are needed (designed for AWS App Runner).

## Setup

```bash
npm install
```

## Development

```bash
npm run dev
# starts on http://localhost:3000
```

## Production

```bash
npm run build
npm run preview
```

See the [Nuxt deployment docs](https://nuxt.com/docs/getting-started/deployment) for hosting options.

