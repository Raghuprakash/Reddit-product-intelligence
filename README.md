# Reddit Costco Product Intelligence

A local-first Python data project that analyzes public discussions in **r/Costco** to discover frequently discussed products and generate transparent, evidence-weighted recommendation rankings.

The project collects recent subreddit submissions and comments through the Reddit Data API using PRAW, identifies candidate product phrases, manually validates products, classifies basic positive and negative recommendation signals, and produces product-level ranking tables.

## Project goal

Answer the question:

> Which products receive the strongest recommendation signals in recent public r/Costco discussions?

This project does not claim to identify the “best” Costco products. Instead, it measures discussion and recommendation signals within the selected Reddit corpus, collection period, and ranking rules.

## Current status

**Phase:** Repository setup and local proof of concept.

**Current scope:**

- Data source: public content from `r/Costco`
- Collection method: Reddit Data API through PRAW
- Processing environment: local Python environment
- Output: ranked CSV tables and a Markdown report
- Deployment: none; runs locally
- Dashboard: planned only after the ranking pipeline is complete

## Planned pipeline

```text
r/Costco submissions
        ↓
Comments from selected discussion threads
        ↓
Text cleaning and deduplication
        ↓
Candidate product-phrase discovery
        ↓
Manual product validation and alias creation
        ↓
Product mention matching
        ↓
Positive / negative / neutral signal labeling
        ↓
Evidence-weighted product rankings
        ↓
CSV output and Markdown report
```

## Project structure

```text
reddit-costco-product-intelligence/
├── README.md
├── .gitignore
├── .env.example
├── requirements.txt
├── config/
│   └── validated_products.csv
├── src/
│   ├── collect_costco_posts.py
│   ├── collect_comments.py
│   ├── discover_products.py
│   └── rank_products.py
├── data/
│   └── sample/
│       └── sample_comments.csv
├── output/
│   └── .gitkeep
└── notebooks/
    └── exploration.ipynb
```

## Tools

- Python
- PRAW for Reddit Data API access
- Pandas for data cleaning, transformation, and aggregation
- PyYAML or CSV configuration files for product aliases
- Regular expressions for initial product matching
- Git and GitHub for version control
- VS Code for development

Optional future tools:

- RapidFuzz for approximate product-name matching
- PyArrow and Parquet for efficient local storage
- Streamlit and Plotly for an interactive dashboard
- Pytest for automated tests
- Docker for reproducible execution

## Setup

### 1. Clone the repository

```bash
git clone [https://github.com/YOUR_GITHUB_USERNAME/reddit-costco-product-intelligence.git](https://github.com/YOUR_GITHUB_USERNAME/reddit-costco-product-intelligence.git)
cd reddit-costco-product-intelligence
```

### 2. Create and activate a virtual environment

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

Windows Command Prompt:

```bat
python -m venv .venv
.venv\Scripts\activate.bat
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Reddit API credentials

Copy the environment-variable template:

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS or Linux:

```bash
cp .env.example .env
```

Open `.env` and add your own Reddit application credentials:

```text
REDDIT_CLIENT_ID=your_client_id
REDDIT_CLIENT_SECRET=your_client_secret
REDDIT_USER_AGENT=python:reddit-costco-product-intelligence:v0.1 (by u_your_reddit_username)
```

Do not commit `.env` to GitHub. It is excluded through `.gitignore`.

## Planned workflow

### 1. Collect r/Costco submissions

The collector retrieves a controlled sample of recent and/or high-engagement submissions from `r/Costco`.

```bash
python -m src.collect_costco_posts
```

Expected local output:

```text
data/raw/costco_posts.parquet
```

### 2. Collect comments

The comment collector retrieves comments from selected candidate threads and saves them locally.

```bash
python -m src.collect_comments
```

Expected local output:

```text
data/raw/costco_comments.parquet
```

### 3. Discover product candidates

The product-discovery script extracts frequent phrases from titles, posts, and comments. Candidate phrases are manually reviewed to distinguish products from generic discussion terms.

```bash
python -m src.discover_products
```

Expected local output:

```text
output/product_candidates.csv
```

### 4. Validate products and aliases

Review the candidate list and create or update:

```text
config/validated_products.csv
```

Example format:

```csv
product_id,brand,product_name,category,aliases
kirkland_protein_bars,Kirkland Signature,Kirkland Protein Bars,snacks,"kirkland protein bars|kirkland bars"
bitchin_sauce,Bitchin' Sauce,Bitchin' Sauce,dips,"bitchin sauce|bitchin' sauce"
bare_chicken_nuggets,Bare,Bare Chicken Nuggets,frozen_food,"bare chicken nuggets|bare nuggets"
```

### 5. Rank products

The ranking script matches validated product aliases in the Reddit corpus, assigns basic recommendation signals, aggregates evidence, and writes ranked results.

```bash
python -m src.rank_products
```

Expected local output:

```text
output/costco_product_rankings.csv
output/costco_product_report.md
```

## Ranking methodology

The first version uses a transparent score:

\[
\text{Recommendation Score} =
2(\text{positive mentions})
- 2(\text{negative mentions})
+ \log(1 + \text{nonnegative comment-score total})
+ 0.5(\text{unique source threads})
\]

The score rewards:

- Positive product recommendations
- Fewer negative product signals
- Comments with positive community engagement
- Evidence spread across multiple discussions

The scoring method will evolve as the project improves. All changes will be documented.

## Example output schema

The final ranking table will include columns like:

| Column | Description |
|---|---|
| `rank` | Product rank within the analyzed corpus |
| `product_name` | Canonical manually validated product name |
| `brand` | Product brand, if known |
| `category` | Product category |
| `total_mentions` | Number of matched product mentions |
| `positive_mentions` | Mentions classified as positive |
| `negative_mentions` | Mentions classified as negative |
| `unique_threads` | Number of unique Reddit threads discussing the product |
| `weighted_comment_score` | Aggregate engagement signal from matched comments |
| `recommendation_score` | Evidence-weighted product ranking score |
| `evidence_level` | Low, medium, or high based on corpus coverage |

Example only—values below are synthetic:

| Rank | Product | Category | Positive | Negative | Threads | Recommendation score |
|---:|---|---|---:|---:|---:|---:|
| 1 | Kirkland Protein Bars | Snacks | 64 | 7 | 28 | 95.2 |
| 2 | Bitchin’ Sauce | Dips | 51 | 4 | 19 | 77.8 |
| 3 | Bare Chicken Nuggets | Frozen food | 42 | 8 | 22 | 64.1 |

## Data and privacy

- This project uses public Reddit content accessed through the Reddit Data API.
- API credentials are stored locally in `.env` and are not committed to the repository.
- Raw collected data is excluded from version control.
- The repository contains code, configuration templates, synthetic sample data, and documentation only.
- Usernames are not displayed in project outputs.
- Source content IDs and permalinks may be retained locally for traceability and deletion/update handling.
- This project is for personal learning and portfolio purposes.

## Limitations

- Reddit is not a representative sample of all Costco shoppers.
- Reddit votes do not equal product quality, sales, safety, or consumer satisfaction.
- Results depend on the selected subreddit, collection dates, retrieved threads, and product alias dictionary.
- Product availability varies by Costco warehouse, location, season, and time.
- A product mention may be ambiguous or incorrectly matched.
- The initial keyword-based sentiment method can misclassify sarcasm, negation, comparisons, and complex opinions.
- Rankings are discussion signals, not official Costco recommendations or purchasing advice.

## Future improvements

- Add incremental collection and data-quality checks.
- Improve phrase extraction and product matching.
- Add approximate alias matching using RapidFuzz.
- Add product categories and category-specific rankings.
- Add aspect-level analysis, such as price, taste, quality, value, durability, and availability.
- Add unit tests and GitHub Actions.
- Store local outputs in Parquet and query them with DuckDB.
- Build a local Streamlit dashboard after the core pipeline is validated.
- Add time-series trends for product recommendation momentum.

## License

This project is licensed under the MIT License. See `LICENSE` for details.