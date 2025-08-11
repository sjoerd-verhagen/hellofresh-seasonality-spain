# 🌱📈 Digging into HelloFresh’s Recipe Data for Fresh, Local, Plant-Forward Ideas

**Author:** Sjoerd Verhagen  
**Tools used:** SQL • Python • Tableau  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)


---

## TL;DR 



---

## 🎯 Background & Project

Barcelona has a growing food-tech scene, and one company I’ve known for years is **HelloFresh**. As a teenager, I learned to cook with their meal kits. At the time, my mother was busy with her studies and my dad isn’t too keen on cooking. So we opted for their subscription, I had first choice out of the three recipes that arrived each week. Back then I didn’t think much about my relationship with food, but it taught me the basics I still use today.

For this project, I wanted to explore how HelloFresh could innovate in line with today’s trends, especially **local sourcing** and **environmental impact**. While they already label some recipes as “seasonal,” I wanted to take it further: What would it look like to analyse their recipe database and explore new possibilities for more local, plant-forward menus?

I found an online HelloFresh recipe database for every country where the company operates. For **HelloFresh Spain**, the vegetarian category alone spanned about **64 pages** (about 12 recipes per page). Direct scraping from the site proved tricky:  the platform is designed to sell recipes, not make them easy to download, to scrape the data I found this hybrid approach:
- A Python script to automatically pull all recipe links per page, massively speeding up the process.
- My Web Scraper automatically did the rest, collecting my recipes and used ingredients

After scraping and combining the CSV files, I removed duplicates, cleaned ingredient lists (stripping out units/quantities), and ended up with **236 unique vegetarian recipes** from HelloFresh Spain ready for analysis.


---

## 🔍 Key Questions

**Part 1: Descriptive statistics**
- In what months are different fruits, herbs, and vegetables in season in Spain?
- What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes? 

**Part 2: Exploring new possibilities for more local, plant-forward menus**
- What % of recipes are seasonal each month? What is the Seasonality Trend?
- Ingredient drivers 1: ingredients most often appear out of season? (chance to change) 
- Ingredient drivers 2: Which are the ‘forgotten’ vegetables? Where is there a lot of potential? 


---

## 🛠️ Tools Used

1. Open Web Scraper for collecting rental data
2. Python for cleaning and preparing the data
3. Google Sheets – quick quality checks on the scraped data
4.  SQL for queries and data analysis
5. Tableau for creating clear visual insights and dashboard

---

## Part 1 – Cleaning the Raw Data

<details>
  <summary>Step 1.1 – "In what months are different fruits, herbs, and vegetables in season in Spain?"

</summary

**Step overview**
For the first step, 

</details> <details> <summary>Step 1.2 – What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes?</summary>

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


## Part 2 – Exploring new possibilities for more local, plant-forward menus (SQL)

<details>
  <summary>Step 2.1 - Descriptive Statistics </summary

**Step overview**

In this step, 
</details>

<details>
  <summary>Step 2.2 – Which districts give the best value for money (€/m²)? </summary

**Step overview**

In this step, 

</details>

<details>
  <summary>Step 2.3 – Where is the largest rental supply? </summary


</details> <details>
  <summary>Step 2.6 – Conclusion </summary
</details>

## What I learned (and Challenges I faced)

