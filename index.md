# 🌱📈 Digging into HelloFresh Spains’s Recipe Data for Fresh, Local, Plant-Forward Ideas

**Author:** Sjoerd Verhagen  
**Tools used:** SQL • Python • Tableau  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)


---

## TL;DR 



---

## 🎯 Background & Project

Barcelona has a growing food-tech scene, and one company I’ve known for years is **HelloFresh**. As a teenager, I learned to cook with their meal kits. At the time, my mother was busy with her studies and my dad isn’t too keen on cooking. So we opted for their subscription, I had first choice out of the three recipes that arrived each week. Back then I didn’t think much about my relationship with food, but it taught me the basics I still use today.

For this project, I wanted to explore how HelloFresh could innovate in line with today’s trends, especially **local sourcing** and **environmental impact**. While they already label some recipes as “seasonal,” I wanted to take it further: What would it look like to analyse their recipe database and explore new possibilities for more local, plant-forward menus?

I found an online HelloFresh recipe database for every country where the company operates. For **HelloFresh Spain**, the vegetarian category alone spanned about **64 pages** (about 12 recipes per page). Many of these pages were doubles, so I looked at that whilst scraping.Direct scraping from the site proved tricky:  the platform is designed to sell recipes, not make them easy to download, to scrape the data I found this hybrid approach:
- A Python script to automatically pull all recipe links per page, massively speeding up the process.
- My Web Scraper automatically did the rest, collecting my recipes and used ingredients
- Afterwards I cleaned the doubles in the data cleaning.

After scraping and combining the CSV files, I removed duplicates, cleaned ingredient lists (stripping out units/quantities), and ended up with **236 unique vegetarian recipes** from HelloFresh Spain ready for analysis.


---

## 🔍 Key Questions

**Part 1: Descriptive statistics**
- Produce Seasonality in Spain
- What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes? 

**Part 2: Exploring new possibilities for more local, plant-forward menus**
- 2.1 What % of recipes are seasonal each month? What is the Seasonality Trend? (which months could use some love?)
- 2.2 Freshness Index: Spotting Overused and Underused Ingredients
- 2.3 Which are the ‘forgotten’ vegetables? Where is there a lot of potential? 


---

## 🛠️ Tools Used

1. Open Web Scraper for collecting rental data
2. Python for cleaning and preparing the data
3. Google Sheets – quick quality checks on the scraped data
4.  SQL for queries and data analysis
5. Tableau for creating clear visual insights and dashboard

---

## Part 1 – Cleaning the Raw Data

</details> <details> <summary>Step 1.1 – Produce Seasonality in Spain </summary>

**Step overview**

For this step, I used Greenpeace’s _La Guía de las Frutas y Verduras de Temporada_, a file showing which fruits and vegetables are in season in Spain throughout the year. I converted the PDF into a CSV listing each vegetable, the months, and whether it is in season or not. The dataset contains **74** products in total.


with **5** of them in season year-round - ajo, cebolla, patata, plátano and zanahoria. The median was **7** months in season, with produce such as tomate (tomato), Brócoli and fresas (strawberries).

This graph shows the number of fresh produce items in season throughout the year in Spain. Interestingly, summer months like **julio** and **agosto** have the fewest fresh items in season, with only 34 and 30 respectively. In contrast, otoño (autumn) and invierno (winter) have the most variety: **octubre** has the highest number with 58 products in season, followed by **noviembre** with 52. When looking at the averages per season, **otoño** has the most produce in season, followed by **invierno**, then primavera (spring), and finally verano (summer) with the least variety.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/When-in-season2.png" alt="In what months are different fruits, herbs, and vegetables in season in Spain?" width="800">

</details> <details> <summary>Step 1.3 – What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes?</summary>


**Step overview**

In this step, I cleaned and matched ingredient names from the seasonality table with those in the recipes table, ensuring consistent formatting by lowercasing and trimming spaces. Then, I counted how many distinct recipes each ingredient appears in to find the most common ingredients. Finally, I calculated the percentage of total recipes that include each ingredient to show its relative frequency.

```sql
WITH clean_seasonality AS (
    SELECT
        LOWER(TRIM(REPLACE(producto, ' (merged)', ''))) AS producto_clean,
        month,
        in_season
    FROM public."11 aug - seasonality"
),

matched_recipes AS (
    SELECT DISTINCT
        LOWER(TRIM(cs.producto_clean)) AS ingredient,
        r."web-scraper-order" AS recipe_id
    FROM public."11 aug - recipes exploded" r
    JOIN clean_seasonality cs
        ON LOWER(TRIM(r."Ingredients")) = LOWER(TRIM(cs.producto_clean))
),

total_recipes AS (
    SELECT COUNT(DISTINCT "web-scraper-order") AS total_count
    FROM public."11 aug - recipes exploded"
)

SELECT
    mr.ingredient,
    COUNT(DISTINCT mr.recipe_id) AS unique_recipe_count,
    ROUND( (COUNT(DISTINCT mr.recipe_id)::decimal / tr.total_count) * 100, 2) AS percent_of_total_recipes
FROM matched_recipes mr
CROSS JOIN total_recipes tr
GROUP BY mr.ingredient, tr.total_count
ORDER BY unique_recipe_count DESC
LIMIT 10;
```

| ingredient | unique_recipe_count | percent_of_total_recipes |
|------------|---------------------|--------------------------|
| cebolla    |                 118 |                      50% |
| calabacín  |                  48 |                   20.34% |
| zanahoria  |                  47 |                   19.92% |
| tomate     |                  27 |                   11.44% |
| limón      |                  26 |                   11.02% |
| patata     |                  22 |                    9.32% |
| perejil    |                  22 |                    9.32% |
| lima       |                  20 |                    8.47% |
| albahaca   |                  20 |                    8.47% |
| berenjena  |                  17 |                     7.2% |

_Cebolla_ (onion) leads by a wide margin, appearing in 50% of recipes. Next are _calabacín_ (zucchini) at 20.34% and _zanahoria_ (carrot) at 19.92%, showing their strong presence. _Tomate_ (tomato) and _limón_ (lemon) feature in about 11% each. Herbs also have a solid role, with _perejil_ (parsley) in 9.32% and _albahaca_ (basil) in 8.47% of recipes. 

</details>

## Part 2 – Exploring new possibilities for more local, plant-forward menus 

<details>
  <summary>Step 2.1 - How seasonal are HelloFresh recipes each month? Tracking trends and spotting low-season months </summary


**Step overview**

In this step, I measure the percentage of in-season ingredients in HelloFresh recipes for each month. The query matches every fresh ingredient from the recipes with the seasonality table to check if it’s in season that month.

It calculates both:
- the overall monthly percentage of in-season ingredients across all recipes
- the average percentage of in-season ingredients per recipe

The output shows, per month: total fresh ingredient mentions, how many are in season, the overall percentage in season, and the recipe-level average percentage in season.

```sql
WITH clean_seasonality AS (
  SELECT
    LOWER(TRIM(REPLACE(producto, ' (merged)', ''))) AS producto_clean,
    LOWER(TRIM(month)) AS month,
    in_season
  FROM public."11 aug - seasonality"
),

recipe_ingredients AS (
  SELECT
    r."web-scraper-order"          AS recipe_id,
    LOWER(TRIM(r."Ingredients"))    AS ingredient
  FROM public."12 aug - recipes exploded" r
),

-- only keep recipe ingredient rows that are fresh produce (appear in seasonality at any month)
fresh_recipe_ingredients AS (
  SELECT ri.recipe_id, ri.ingredient
  FROM recipe_ingredients ri
  JOIN (SELECT DISTINCT producto_clean FROM clean_seasonality) s
    ON ri.ingredient = s.producto_clean
),

-- totals per month counting each ingredient mention once per recipe
monthly_totals AS (
  SELECT
    cs.month,
    COUNT(*) AS total_ingredient_mentions,
    COUNT(*) FILTER (WHERE cs.in_season = 1) AS total_in_season_mentions
  FROM fresh_recipe_ingredients ri
  JOIN clean_seasonality cs
    ON ri.ingredient = cs.producto_clean
  GROUP BY cs.month
),

-- percent in season per recipe then averaged per month
matched_per_recipe_month AS (
  SELECT
    ri.recipe_id,
    cs.month,
    COUNT(*) FILTER (WHERE cs.in_season = 1)    AS ingredients_in_season,
    COUNT(*)                                    AS total_ingredients
  FROM fresh_recipe_ingredients ri
  JOIN clean_seasonality cs
    ON ri.ingredient = cs.producto_clean
  GROUP BY ri.recipe_id, cs.month
),

avg_percent_per_recipe AS (
  SELECT
    month,
    ROUND(AVG((ingredients_in_season::decimal / NULLIF(total_ingredients,0)) * 100), 2) AS avg_percent_per_recipe
  FROM matched_per_recipe_month
  GROUP BY month
)

SELECT
  mt.month,
  mt.total_ingredient_mentions      AS total_ingredients,
  mt.total_in_season_mentions       AS total_ingredients_in_season,
  ROUND((mt.total_in_season_mentions::decimal / NULLIF(mt.total_ingredient_mentions,0)) * 100, 2)
    AS percent_in_season_overall,
  ap.avg_percent_per_recipe
FROM monthly_totals mt
LEFT JOIN avg_percent_per_recipe ap USING (month)
ORDER BY CASE mt.month
  WHEN 'enero' THEN 1
  WHEN 'febrero' THEN 2
  WHEN 'marzo' THEN 3
  WHEN 'abril' THEN 4
  WHEN 'mayo' THEN 5
  WHEN 'junio' THEN 6
  WHEN 'julio' THEN 7
  WHEN 'agosto' THEN 8
  WHEN 'septiembre' THEN 9
  WHEN 'octubre' THEN 10
  WHEN 'noviembre' THEN 11
  WHEN 'diciembre' THEN 12
END;
```

| "month"      | "total_ingredients" | "total_ingredients_in_season" | "percent_in_season_overall" | "avg_percent_per_recipe" |
|--------------|---------------------|-------------------------------|-----------------------------|--------------------------|
| "enero"      | 531                 | 353                           | 66.48                       | 68.71                    |
| "febrero"    | 531                 | 359                           | 67.61                       | 70.15                    |
| "marzo"      | 531                 | 321                           | 60.45                       | 62.92                    |
| "abril"      | 531                 | 359                           | 67.61                       | 70.15                    |
| "mayo"       | 531                 | 448                           | 84.37                       | 85.92                    |
| "junio"      | 531                 | 441                           | 83.05                       | 83.45                    |
| "julio"      | 531                 | 387                           | 72.88                       | 71.83                    |
| "agosto"     | 531                 | 383                           | 72.13                       | 70.99                    |
| "septiembre" | 531                 | 466                           | 87.76                       | 87.53                    |
| "octubre"    | 531                 | 523                           | 98.49                       | 98.02                    |
| "noviembre"  | 531                 | 441                           | 83.05                       | 82.99                    |
| "deciembre"  | 531                 | 381                           | 71.75                       | 72.72                    |

**ADD GRAPH** "What % of recipes are seasonal in each month"

We see that winter has the lowest percentage of seasonal ingredients, with spring and summer only slightly higher. Autumn stands out with 89.51% of recipes in season. Looking at individual months, March is the lowest at 62.92%, followed by January, February, and April, all hovering between 69% and 70%. A similar dip appears in summer, with July and August both around 71%.


</details>
<details>
  <summary>Step 2.2 – Freshness Index: Spotting Overused and Underused Ingredients </summary


**Step overview**

In this step,  I used a _freshness index_ to compare how often ingredients appear in recipes with how often they are actually in season. By plotting the percentage of the year they’re fresh against their usage percentage, we can see clear patterns. I removed one outlier, onion, which is always in season and covers 50% of recipes.

**Overused** – Ingredients used frequently but not in season for much of the year.
For example, _calabacín_ (courgette) appears in 20% of recipes but is only in season for half the year. _Tomato_ is fresh seven months per year yet is a regular feature. Others with a high freshness gap include _albahaca_ (basil), _lima_ (lime), and _berenjena_ (aubergine) – each used 7–8% of the time but only in season for four to six months.

**Underused despite high availability** – Ingredients that are in season most of the year but rarely used.
Some, like _ajo_ (garlic), _patata_ (potato), and _zanahoria_ (carrot), are fresh year-round and used often. But others are surprisingly underrepresented, such as _pack choi_ (in season 10 months but only used in one recipe) and _rábano_ (radish, in season 9 months, also just one recipe). 

</details>

<details>
  <summary>Step 2.3 – Which are the ‘forgotten’ vegetables? Where is there a lot of potential? </summary

**Step overview**
In this step...

Forgotten vegetables ccan of course be seen as vegetables which are never used in this batch of recipes. Could be for good reason, where people dont know these vegetables so its les easy to market it as easy and tasty meals. One way of looking at it is seeing forgotten as never used, and vegetables used in a lot of recipes (known, quite common). But other, maybe more intersting for our purposes here, is vegetabels that are currently in use in the recipes, but that are currently underused, especially looking at their seasonality profile. Plus, this means that there are already suppliers known to hellofresh to supply these vegeatbles, making it a quick win and something that one can market as "in season". 



</details> <details>
  <summary>Step 2.6 – Conclusion </summary
</details>

## What I learned (and Challenges I faced)

