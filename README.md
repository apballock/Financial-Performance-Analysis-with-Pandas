# Financial Performance Analysis
### Python · Pandas · Data Wrangling · Exploratory Analysis
 
A end-to-end data analysis project built on a real-world financial sales dataset spanning 6 countries, 5 business segments, and 688 transactions. The project covers the full analytical workflow — from raw data ingestion and quality assessment through aggregation, joins, and dimensional reshaping — structured across two progressive levels.
 
---
 
## Dataset
 
**Source:** Financial Sample Dataset — global sales records across Canada, France, Germany, Mexico, USA, and United Kingdom.
 
**Raw state:** 688 rows × 22 columns, with mixed-language column names (Spanish/English), European-format numeric strings (`10,56 €`), and 53 structurally null values in the discount dimension.
 
**Clean state:** 688 rows × 26 columns after renaming, type conversion, null treatment, and derived feature engineering.
 
---
 
## Project Structure
 
```
financial-performance-analysis/
│
├── data/
│   ├── financials.csv               # Raw dataset
│   └── financials_clean.csv         # Processed dataset
│
├── nivel1_intro_pandas.ipynb        # Data ingestion, cleaning, feature engineering
└── nivel2_groupby_joins_reshape.ipynb  # Aggregation, joins, and reshaping
```
 
---
 
## Level 1 — Data Ingestion & Feature Engineering
 
**Notebook:** `nivel1_intro_pandas.ipynb`
 
### Data Quality Assessment
 
On load, the dataset presented three quality issues that required deliberate treatment:
 
**1. Column naming inconsistency** — 14 of 22 columns carried Spanish-language prefixes (`Suma de Sales`, `Año`, `Trimestre`) inherited from a Power BI export. All were renamed to standard English for downstream compatibility.
 
**2. Numeric columns stored as strings** — `Avg Sale per Unit` and `Total COGS` were encoded with European decimal formatting and currency symbols (`10,56 €`, `8.283 €`). Both columns were cleaned via `.str.replace()` chaining and cast to `float64`.
 
**3. Structurally null values** — `Discount Band` showed 53 null entries. Rather than imputing or dropping, the nulls were investigated first: all 53 rows had `Discounts = 0.0`, confirming the absence of a discount band was meaningful, not missing. Values were filled with `'None'` to preserve the semantic distinction.
 
### Feature Engineering
 
Four analytical columns were derived from existing data:
 
| Column | Logic | Purpose |
|---|---|---|
| `Profit Margin %` | `(Profit / Sales) * 100` | Normalized profitability metric |
| `Profitable` | `np.where(Profit > 0, 'Yes', 'No')` | Binary profitability flag |
| `Performance` | Custom `apply()` function on margin | 4-tier categorical classification |
| `Revenue Category` | `pd.qcut()` on Sales, 4 quantiles | Quartile-based revenue segmentation |
 
**Key finding:** 91% of transactions (625/688) are profitable. However, margin distribution is heavily skewed — the top quartile (`Very High`) starts above the 75th percentile of profit, masking significant variance within the profitable segment.
 
---
 
## Level 2 — Aggregation, Joins & Reshaping
 
**Notebook:** `nivel2_groupby_joins_reshape.ipynb`
 
### Groupby & Crosstab
 
Segment-level aggregation using `.groupby().agg()` revealed a structural tension in the data:
 
| Segment | Total Sales | Total Profit | Avg Margin |
|---|---|---|---|
| Government | 52.5M | 11.4M | 29.4% |
| Small Business | 42.4M | 4.1M | 9.7% |
| Enterprise | 19.6M | -614K | -3.1% |
| Midmarket | 2.4M | 660K | 27.7% |
| Channel Partners | 1.8M | 1.3M | 73.0% |
 
**Notable:** Enterprise is the third-largest segment by revenue but operates at a net loss. Channel Partners is the smallest by volume but the most efficient by margin — 73% average. This kind of divergence between revenue rank and profitability rank is a classic signal for pricing or cost structure review.
 
`pd.crosstab()` was used to produce a country × segment sales matrix, confirming that Government and Small Business dominate across all geographies, with France leading Government sales at 12.1M.
 
### Merge & Concat
 
Demonstrated the two core DataFrame combination patterns:
 
- **`pd.merge()` (inner join):** Combined a `sales` table with a `products` table on `product_id` to calculate `total_revenue = units_sold × unit_price` — a pattern equivalent to a SQL JOIN used in virtually every revenue reporting pipeline.
- **`pd.merge()` (left join):** Extended the products table with a product carrying no sales history (`Carretera`). The left join preserved the product with `NaN` across all sales columns — the standard pattern for identifying inactive products, lapsed customers, or unfulfilled inventory.
- **`pd.concat()`:** Stacked two annual sales DataFrames (2023 + 2024) vertically, the typical pattern when consolidating period-over-period files from the same source schema.
### Reshaping
 
Demonstrated the wide ↔ long transformation cycle using a country × quarterly sales dataset:
 
- **`melt()`:** Collapsed four quarterly columns (`Q1_Sales` → `Q4_Sales`) into a single `Quarter` column with a corresponding `Sales` value — the format required by most visualization libraries and BI tools for time-series plotting.
- **`pivot()`:** Reversed the operation, reconstructing the wide matrix with countries as index and quarters as columns.
- **Total column via `axis=1`:** Summed across columns (horizontally) to produce annual totals per country. USA led at 90K across all quarters; Canada was lowest at 43K.
---
 
## Stack
 
- Python 3.x
- Pandas
- NumPy
- Jupyter Notebook
---
 
## Key Takeaways
 
Segment revenue rank and profitability rank do not align — Enterprise generates 3× more revenue than Channel Partners but runs at a loss, while Channel Partners operates at 73% margin. In a real business context, this would trigger a review of Enterprise pricing strategy or cost allocation.
 
The null treatment in `Discount Band` is an example of investigation-driven imputation: the decision to fill with `'None'` rather than drop or use a statistical fill was justified by the data itself, not assumed.
