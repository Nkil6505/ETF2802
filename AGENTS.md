# AGENTS.md

## Cursor Cloud specific instructions

### Project overview

Static single-page ETF dividend monitoring dashboard (`index.html`) for Hong Kong-listed covered call ETFs (2802.HK, 3416.HK) with HSCEI benchmark. No build step, no package manager, no backend — pure HTML/CSS/JS with Chart.js loaded via CDN.

### Running the application

Serve the workspace root with any static HTTP server:

```
python3 -m http.server 8080
```

Then open `http://localhost:8080`. The app fetches `data/live.json` and `data/history.json` via relative paths, so `file://` protocol will not work — it must be served over HTTP.

### Key notes

- **No lint/test/build commands exist.** There is no `package.json`, no test framework, and no linter configured. Validation is done by loading the page in a browser and confirming it renders.
- **All user data is in `localStorage`.** Keys: `etf2802_history`, `etf2802_port`. Clearing browser storage resets all portfolio/history data.
- **`data/live.json`** and **`data/history.json`** are auto-updated by the GitHub Actions workflow (`.github/workflows/fetch-prices.yml`) using `yfinance`. These files also serve as read-only cache for the frontend when CORS-proxied Yahoo Finance API is unavailable.
- **Theme toggle** (dark/light) is in the top-right corner of the page.
