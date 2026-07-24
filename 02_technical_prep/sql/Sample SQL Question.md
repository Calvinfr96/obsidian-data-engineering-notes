## Prompt
Write a SQL query that calculates the overall Click-Through Rate (CTR) for each campaign_id for the current month. The CTR is defined as the Total Number of Clicks / Total Number of Impressions. Your output should contain three columns: campaign_id, total_impressions, and ctr_percentage. Round the CTR percentage to 2 decimal places, and order the results from highest CTR to lowest.

## Proposed Solution

```sql
-- Step 1: Aggregate impressions for the current month

WITH monthly_impressions AS (

    SELECT 

        campaign_id,

        COUNT(impression_id) AS total_impressions

    FROM ad_impressions

    WHERE timestamp >= '2026-07-01' AND timestamp < '2026-08-01'  -- Dynamic/explicit for current month

    GROUP BY campaign_id

),

  

-- Step 2: Aggregate clicks for the current month

monthly_clicks AS (

    SELECT 

        campaign_id,

        COUNT(click_id) AS total_clicks

    FROM ad_clicks

    WHERE timestamp >= '2026-07-01' AND timestamp < '2026-08-01'

    GROUP BY campaign_id

)

  

-- Step 3: Combine them using a LEFT JOIN to preserve campaigns with 0 clicks

SELECT 

    i.campaign_id,

    i.total_impressions,

    -- Handle Integer Division by multiplying by 100.0 (converts to decimal)

    -- Handle Zero Division or NULLs using COALESCE if clicks don't exist

    ROUND(

        (COALESCE(c.total_clicks, 0) * 100.0) / i.total_impressions, 

        2

    ) AS ctr_percentage

FROM monthly_impressions i

LEFT JOIN monthly_clicks c ON i.campaign_id = c.campaign_id

ORDER BY ctr_percentage DESC;
```
### Breakdown
1. **Date Filtering Performance**: Instead of using functions like MONTH(timestamp) = 7 on a WHERE clause, using a date range (>= '2026-07-01') allows the database to use index scanning (Sargable expression). Running a function on a column forces a slow full table scan.
2. **COALESCE(..., 0)**: If a campaign had impressions but zero clicks, the LEFT JOIN yields a NULL for clicks. COALESCE converts that NULL to 0, preventing your math from breaking.

3. **100.0:** Multiplying by a decimal numeric literal implicitly casts the numerator, flawlessly bypassing integer division rules in PostgreSQL/Hive/Redshift.