# SOCIAL-MEDIA-ANALYSIS
This project explores influencer data across Instagram, TikTok, and YouTube using SQL. It includes data cleaning, quality checks, platform performance comparison, audience growth analysis, and top-tier influencer identification to evaluate social media dominance and engagement trends.
# 📊 Social Media Analysis

![Image alt](https://github.com/DataDrivenMaryam/SOCIAL-MEDIA-ANALYSIS/blob/main/Futuristic%20Social%20Media%20Analytics%20Banner.png)
---

##  Project Overview

This project analyzes influencer performance across **Instagram, TikTok, and YouTube** using SQL.

The analysis focuses on:
- Influencer distribution
- Platform performance comparison
- Audience growth trends
- Top-tier influencer identification
- Category dominance

---

##  Tools Used
- MySQL
- SQL
- GitHub

---

#  Phase 1 — Database Setup

Work:

Create project database

Create raw tables (by platform and/or month)

Define data types (text vs numeric)

Add fields for platform and snapshot month

Load all CSVs

Deliverables:

SQL scripts for database + table creation + loading

Row count per table

Written explanation of table design choices

#  Phase 2 — Standardization & Integration

Work:

Standardize column names across datasets

Normalize platform naming conventions

Build a consolidated influencers table using UNION ALL

Preserve platform + month context in the unified dataset

Deliverables:

SQL script creating the consolidated influencers table/view

Explanation of how monthly datasets were combined

Validation query showing record totals by platform and month
Comment:
This was handled during data cleaning. Columns representing the same metric were made consistent so analysis and comparisons work reliably (e.g., followers/subscribers formatted consistently).



#  Phase 3 — Data Understanding & Quality Checks
Key Data Limitations (Observed)

Influencer naming inconsistencies (mixture of characters and alphabets) can complicate grouping/joins

Followers/views may be missing, zero, or inconsistent

Influencers appear across multiple months (valid), but can affect duplicate logic if not handled carefully

#  Phase 4 — Core Social Media Analysis Tasks
1. Total number of influencers by platform

SELECT platform, COUNT(*) AS total_influencers
FROM (
    SELECT sn, 'ig' AS platform FROM ig
    UNION ALL
    SELECT sn, 'tiktok' AS platform FROM tiktok
    UNION ALL
    SELECT sn, 'youtube' AS platform FROM youtube
) AS all_platforms
GROUP BY platform;
# Comment:
This combines all platforms into a single dataset using UNION ALL and counts influencers per platform.
UNION ALL is faster and appropriate because the datasets represent different platforms.

# 2) Influencer distribution by country

SELECT country, COUNT(*) AS influence_distribution
FROM (
    SELECT SN, country, 'ig' AS platform FROM ig
    UNION ALL
    SELECT SN, country, 'YouTube' AS platform FROM YouTube
) AS all_influencers_by_dist
GROUP BY country;
# Comment:

3) Average followers/subscribers by platform
This counts influencers per country across platforms included in the union to understand geographic spread.
SELECT Platform, AVG(subscriber_count) AS avg_subscriber
FROM (
    SELECT Followers_million AS subscriber_count, 'ig' AS platform FROM ig
    UNION ALL
    SELECT 
        (subcribers_count_Thousand * 1000 + Subscribers_count_Million * 1000000) AS subscriber_count,
        'tiktok' AS platform
    FROM tiktok
    UNION ALL
    SELECT Subscribers_count AS subscriber_count, 'youtube' AS platform FROM youtube
) AS all_platforms
GROUP BY Platform;

#Comment:
This standardizes audience size metrics across platforms into a single numeric field for accurate averaging.

#  Phase 5 — Platform Comparison Analysis
Which platform has the highest average audience size?
SELECT
  platform,
  AVG(audience_millions) AS avg_audience_millions,
  COUNT(*) AS total_rows
FROM (
  SELECT 'ig' AS platform, Followers_million * 1.0 AS audience_millions
  FROM ig
  WHERE Followers_million IS NOT NULL

  UNION ALL

  SELECT 'tiktok' AS platform,
         (Subscribers_count_Million * 1.0) + (subcribers_count_Thousand / 1000.0) AS audience_millions
  FROM tiktok
  WHERE Subscribers_count_Million IS NOT NULL
     OR subcribers_count_Thousand IS NOT NULL

  UNION ALL

  SELECT 'youtube' AS platform, Subscribers_count * 1.0 AS audience_millions
  FROM youtube
  WHERE Subscribers_count IS NOT NULL
) AS ALL_PLATFORM
WHERE audience_millions IS NOT NULL
GROUP BY platform
ORDER BY avg_audience_millions DESC;


Comment:
This query compares platforms by standardizing audience size into “millions,” filtering nulls, then ranking platforms by average audience.

![image alt]<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/1e8cfce9-e7c0-4c25-97f9-fc94f90c8d75" />

#  Phase 6 — Time-Based Growth Analysis
Instagram average audience growth across months
SELECT
    curr.platform,
    curr.month AS current_month,
    prev.month AS previous_month,
    curr.avg_audience_millions AS current_avg_audience_millions,
    prev.avg_audience_millions AS previous_avg_audience_millions,
    ROUND(curr.avg_audience_millions - prev.avg_audience_millions, 3) AS avg_audience_growth_millions
FROM (
    SELECT 'ig' AS platform, Month, AVG(Followers_million) AS avg_audience_millions
    FROM ig
    GROUP BY Month
) curr
JOIN (
    SELECT 'ig' AS platform, Month, AVG(Followers_million) AS avg_audience_millions
    FROM ig
    GROUP BY Month
) prev
ON curr.platform = prev.platform
AND (
       (curr.month = 'Nov'      AND prev.month = 'Sept')
    OR (curr.month = 'December' AND prev.month = 'Nov')
    OR (curr.month = 'Latest'   AND prev.month = 'December')
)
ORDER BY curr.month;
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/dd1a8797-c7aa-4f9c-8ce0-f6db1977b923" />

Comment:
This compares monthly averages month-to-month to estimate platform-level growth over time.

TikTok average audience growth across months
SELECT
    curr.platform,
    curr.month AS current_month,
    prev.month AS previous_month,
    ROUND(curr.avg_audience_millions, 3) AS current_avg_audience_millions,
    ROUND(prev.avg_audience_millions, 3) AS previous_avg_audience_millions,
    ROUND(curr.avg_audience_millions - prev.avg_audience_millions, 3) AS avg_audience_growth_millions
FROM (
    SELECT 'tiktok' AS platform, Month,
           AVG((Subscribers_count_Million * 1.0) + (subcribers_count_Thousand / 1000.0)) AS avg_audience_millions
    FROM tiktok
    WHERE Subscribers_count_Million IS NOT NULL
       OR subcribers_count_Thousand IS NOT NULL
    GROUP BY Month
) curr
JOIN (
    SELECT 'tiktok' AS platform, Month,
           AVG((Subscribers_count_Million * 1.0) + (subcribers_count_Thousand / 1000.0)) AS avg_audience_millions
    FROM tiktok
    WHERE Subscribers_count_Million IS NOT NULL
       OR subcribers_count_Thousand IS NOT NULL
    GROUP BY Month
) prev
ON curr.platform = prev.platform
AND (
       (curr.month = 'Nov'      AND prev.month = 'Sept')
    OR (curr.month = 'December' AND prev.month = 'Nov')
    OR (curr.month = 'Latest'   AND prev.month = 'December')
)
ORDER BY curr.month;


Comment:
Same growth logic as Instagram, but TikTok audience is calculated by combining million + thousand subscriber fields.

#  Phase 7 — Compare Influencer Reach vs Platform Averages
Instagram: influencer vs platform average
SELECT
    i.SN AS influencer_id,
    i.Month,
    i.Followers_million AS influencer_audience_millions,
    p.avg_audience_millions AS platform_avg_audience_millions,
    ROUND(i.Followers_million - p.avg_audience_millions, 3) AS difference_from_platform_avg,
    CASE
        WHEN i.Followers_million > p.avg_audience_millions THEN 'Above Platform Average'
        ELSE 'Below Platform Average'
    END AS performance_vs_platform
FROM ig i
JOIN (
    SELECT Month, AVG(Followers_million) AS avg_audience_millions
    FROM ig
    WHERE Followers_million IS NOT NULL
    GROUP BY Month
) p
ON i.Month = p.Month
WHERE i.Followers_million IS NOT NULL
ORDER BY difference_from_platform_avg DESC;

![image alt]<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/703f4953-1ca2-40c2-a404-8e8710bb7341" />

Comment:
This benchmarks each influencer against the monthly platform average and labels performance as above/below average.

(You applied the same logic for YouTube and TikTok — great consistency.)

#  Phase 8 — Conditional Logic & Segmentation
Identify top-tier influencers (audience ≥ 10M)
SELECT
    platform,
    Month AS month,
    influencer_id,
    audience_millions
FROM (
    SELECT 'ig' AS platform, Month, SN AS influencer_id, Followers_million * 1.0 AS audience_millions
    FROM ig
    WHERE Followers_million IS NOT NULL

    UNION ALL

    SELECT 'tiktok' AS platform, Month, SN AS influencer_id,
           (Subscribers_count_Million * 1.0) + (subcribers_count_Thousand / 1000.0) AS audience_millions
    FROM tiktok
    WHERE Subscribers_count_Million IS NOT NULL
       OR subcribers_count_Thousand IS NOT NULL

    UNION ALL

    SELECT 'youtube' AS platform, Month, SN AS influencer_id, Subscribers_count * 1.0 AS audience_millions
    FROM youtube
    WHERE Subscribers_count IS NOT NULL
) AS combined
WHERE audience_millions >= 10
ORDER BY audience_millions DESC;
<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/703f4953-1ca2-40c2-a404-8e8710bb7341" />
Comment:
This segments influencers into a “top-tier” group using a clear threshold rule across platforms.

#  Key Insights Generated

Certain platforms host significantly larger average audiences

Influencer growth varies substantially by platform and month

Specific countries consistently dominate influencer reach

A small percentage of influencers control a large share of total audience

#  Skills Demonstrated

Database design

Multi-table integration

Data standardization

Data quality validation

Aggregations & grouping

Window functions

Conditional logic

Business insight translation

