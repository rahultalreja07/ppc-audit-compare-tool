# PPC Audit App

A Nuxt 4 dashboard for daily cross-source data consistency checks across **GPP**, **SEC EDGAR**, and **fund websites**. For each fund (identified by CIK), it surfaces NAV discrepancies, distribution data, shares outstanding, and GPP time-series charts — all in one place.

## Features

- **Comparison Dashboard** — filterable list of funds by date, fund name/CIK, and overall status (`match` / `mismatch` / `error` / `ok`)
- **Detailed comparison view** with four tabs:
  - **NAV** — per-class NAV comparison between SEC and website
  - **Distributions** — gross distribution, servicing fee, and net distribution per class
  - **Shares** — shares issued and percentage of total per class
  - **GPP** — time-series line charts (Price, Dividend, ClassTNA) with expected vs available section validation
- **Typed data layer** — shared TypeScript interfaces for all comparison structures
- **Pluggable data adapter** — currently reads from local `data/` JSON files; swap `server/utils/dataAdapter.ts` for AWS S3 calls to migrate to cloud storage

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Nuxt 4](https://nuxt.com) + [Vue 3](https://vuejs.org) |
| Styling | [Tailwind CSS](https://tailwindcss.com) |
| Charts | [Chart.js](https://www.chartjs.org) + [vue-chartjs](https://vue-chartjs.org) |
| Language | TypeScript |
| Runtime | Node.js (Nitro server) |

## Project Structure

```
app/
  pages/
    index.vue                  # Dashboard list (date + search + status filter)
    comparison/[date]/[cik].vue # Fund detail page (4 tabs)
  components/
    tab/                       # NAV, Distributions, Shares, GPP tab components
    BaseTabs.vue / StatusBadge.vue / ...
  composables/
    useComparisons.ts          # Fetches CIK list for a given date
    useComparisonDetail.ts     # Fetches full detail for a CIK
server/
  api/
    dates.get.ts               # GET /api/dates
    comparisons/[date].get.ts  # GET /api/comparisons/:date
    comparison/[date]/[cik].get.ts # GET /api/comparison/:date/:cik
  utils/
    dataAdapter.ts             # File-system data layer (swap for S3)
types/
  comparison.ts                # Shared TypeScript interfaces
data/                          # Local audit data (git-ignored)
  <YYYY-MM-DD>/
    index.json                 # CIK summary list
    <cik>.json                 # Full comparison detail per fund
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

**`<cik>.json`** — full `ComparisonDetail` object (see `types/comparison.ts` for the complete schema).

The `data/` directory is git-ignored. Place your analysis files there before running the app.

## Setup

Install dependencies:

```bash
npm install
```

## Development Server

Start the development server on `http://localhost:3000`:

```bash
npm run dev
```

## Production

Build the application for production:

```bash
npm run build

# yarn
yarn build

# bun
bun run build
```

Locally preview production build:

```bash
# npm
npm run preview

# pnpm
pnpm preview

# yarn
yarn preview

# bun
bun run preview
```

Check out the [deployment documentation](https://nuxt.com/docs/getting-started/deployment) for more information.
