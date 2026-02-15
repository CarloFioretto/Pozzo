# Commodities Data Pipeline

This project implements a Python pipeline that pulls yesterday's commodities data, computes the top 3 performers, saves the raw data to JSON/CSV, and updates a Notion database with the results.

## Setup

### Prerequisites
- Python 3.8+
- Notion API token
- Notion parent page ID

### Installation
1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. Install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Create a `.env` file in the project root and add your Notion API token and parent page ID:
   ```env
   NOTION_TOKEN=your_notion_api_token_here
   NOTION_PARENT_PAGE_ID=your_notion_parent_page_id_here
   
   # Optional: Override the date (YYYY-MM-DD format)
   # If not set, defaults to yesterday with automatic weekend handling
   # COMMODITIES_DATE=2026-02-13
   ```

## Usage

Run the pipeline:
```bash
python src/commodities_pipeline.py
```

**Date Override Options:**

You can specify a different date using either:

1. **CLI argument** (highest priority):
   ```bash
   python src/commodities_pipeline.py --date 2026-02-13
   ```

2. **Environment variable**:
   ```bash
   COMMODITIES_DATE=2026-02-13 python src/commodities_pipeline.py
   ```

3. **Both (CLI takes precedence)**:
   ```bash
   COMMODITIES_DATE=2026-02-12 python src/commodities_pipeline.py --date 2026-02-13
   ```
   This will use 2026-02-13 (from CLI), not 2026-02-12 (from env var).

**Note**: If `yfinance` fails to fetch data (e.g., due to market holidays or API issues), the script will use mock data for testing purposes. This ensures the pipeline can be tested even when live data is unavailable.

## Outputs

- **Primary Data Files**: Saved in the `data/` directory as `data_YYYY-MM-DD.json` and `data_YYYY-MM-DD.csv`.
- **Raw Data Files**: Also saved as `raw_YYYY-MM-DD.json` and `raw_YYYY-MM-DD.csv` for backwards compatibility.
- **Markdown Report**: Generated in the `data/` directory as `report_commodities_YYYY-MM-DD.md` with detailed insights.
- **Notion Database**: Updated with the latest commodities data and top performers tagged.

### Weekend Handling
If the script runs on a Monday or the previous day was a weekend, it automatically fetches data for the last trading day (Friday). This ensures you always get the most recent market data.

## Configuration

The list of commodity tickers and other configurations can be modified in the `src/commodities_pipeline.py` file.

## Notion Database Schema

The Notion database will have the following properties:
- **Ticker**: Title property (e.g., "GC=F", "SI=F")
- **Name**: Rich text property (e.g., "Gold", "Silver")
- **Date**: Date property
- **Price**: Number property
- **Change**: Number property (percent change)
- **Top Performer**: Checkbox property (tagged for top 3 performers)

### Idempotent Upserts
The pipeline uses idempotent upserts to avoid duplicate rows. For each ticker+date combination:
- If an entry already exists, it is updated with the latest data
- If no entry exists, a new one is created

This ensures you can safely run the pipeline multiple times without creating duplicates.

### Top Performers View
To create a filtered view for top performers in Notion:
1. Open the database in Notion
2. Click "+ New" next to existing views
3. Select a view type (e.g., Table or Gallery)
4. Name it "Top Performers"
5. Add filter: "Top Performer" is checked

Note: The Notion API does not support programmatic creation of filtered views, so this must be done manually once after the database is created.

## License

This project is licensed under the MIT License.