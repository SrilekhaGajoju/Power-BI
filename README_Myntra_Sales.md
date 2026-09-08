# Myntra E-Commerce Pricing \& Discount Analysis

A Power BI dashboard analyzing \~52,000 scraped men's jeans listings from Myntra, focused on pricing strategy, discount behavior, and rating credibility.

## Business Questions Answered

1. Which brands offer the deepest average discounts, and does that correlate with product rating?
2. How are products distributed across price tiers (Budget / Mid-Range / Premium / Luxury)?
3. Which are the top 10 highest-rated products, restricted to those with a statistically credible number of reviews (1,000+ ratings)?
4. Is there a visible relationship between discount depth and rating at the brand level? *(scatter plot, same underlying question as #1)*

## Data Source

[Myntra Sales Dataset – Kaggle](https://www.kaggle.com/datasets/skmewati/myntra-sales-dataset) — 52,120 raw rows scraped from Myntra's men's jeans category. Fields: brand name, product description, price, MRP, discount %, rating, and number of ratings.

## Tools Used

Power BI Desktop · Power Query (M) · DAX

## Data Cleaning — Key Findings

Cleaning surfaced two significant, non-obvious data quality issues that would have silently corrupted analysis if left unaddressed:

Exact duplicate rows: 

17,047 of 52,120 rows (32.7%) were identical across every column — almost certainly repeated scraping of the same product pages

Removed via full-row duplicate matching, leaving 35,073 rows

Corrupted discount field:
3,557 rows (10.1%) had a `discount\_percent` value that didn't reconcile with the actual Price/MRP relationship (e.g., a stored 21% discount where the real figure was 71%)	

Replaced the scraped column entirely with a recalculated `Discount(%) = 1 − (Price ÷ MRP)`, verified against all rows

## Data Model

Single flat table (`Products`, 35,073 rows). No relationships required — this dataset is a product-listing snapshot, not transactional data, so no date dimension or star schema applies here (that comes in Project 2).

**Key modeling decisions:**

* Set `price`, `MRP`, and `ratings` to "Average" default summarization instead of "Sum" — summing individual product prices/ratings across a catalog produces a meaningless total, not a usable KPI
* Built a `Brand Has Enough Data` flag column (Yes/No) to exclude the 42 brands with fewer than 20 listings from brand-level comparisons, avoiding misleading averages built on tiny samples

## Key DAX Measures

```dax
Total Products = COUNTROWS(Products)

Average Discount % = AVERAGE(Products\[Discount(%)])

Price Band =
SWITCH(
    TRUE(),
    Products[price] < 1000, "Budget (<Rs.1,000)",
    Products[price] < 2000, "Mid-Range (Rs.1,000-2,000)",
    Products[price] < 5000, "Premium (Rs.2,000-5,000)",
    "Luxury (Rs.5,000+)"
)

Brand Has Enough Data =
IF(
    CALCULATE(COUNTROWS(Products), ALLEXCEPT(Products, Products\[brand\_name])) >= 20,
    "Yes", "No"
)
```

## Findings

* **Price distribution:** Mid-Range (₹1,000–2,000) is the dominant tier at 49.8% of listings, followed by Budget (30.8%), Premium (17.6%), and a small Luxury tail (1.8%) — including outliers as high as ₹54,000.
* **Discount vs. rating:** A weak negative correlation (r = -0.226) exists across the 193 brands with 20+ listings — deeper average discounts are mildly associated with slightly lower ratings, though the relationship is far from strong. The scatter plot shows a loosely scattered cloud rather than a tight trend.
* **Top-rated products (1,000+ ratings only):** Levis dominates the credible top 10, holding 6 of the 10 spots — primarily 511 and 512 Slim Fit variants across different price points and colorways.


## Report Pages

1. **Overview** — KPI cards (Total Products, Avg Discount %, Avg Price, Avg Rating)
2. **Brand Discount Analysis** — Combo chart (Top 15 brands by discount, with rating overlay) + scatter plot (discount % vs. rating, sized by brand product count)
3. **Price Distribution** — Price band breakdown
4. **Top Rated Products** — Filtered table of the 10 highest-rated products with 1,000+ reviews.

