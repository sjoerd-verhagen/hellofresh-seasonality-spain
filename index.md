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

**Part 1: Descriptive Statistics**
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

## Part 1 – Descriptive Statistics

</details> <details> <summary>Step 1.1 – Produce Seasonality in Spain </summary>

**Step overview**

To understand how well HelloFresh recipes align with what is naturally available, the first step is to map out the seasonality of fresh produce in Spain. For this, I worked with _**Greenpeace’s La Guía de las Frutas y Verduras de Temporada**_ [The Seasonal Fruit and Vegetable Guide], which lists the fruits and vegetables that are in season in Spain each month. I converted the PDF into a CSV, with each row showing the product name, the month, and whether it is in season. The dataset covers **74** fresh products in total. 

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/vegs bubbles2.png" width="800">

Out of these, **5** items are available year-round: ajo (garlic), cebolla (onion), patata (potato), plátano (plantain), and zanahoria (carrot). The median availability is **7** months per year, with produce such as tomate (tomato), brócoli (broccoli), and fresas (strawberries) all falling into this middle range. In the chart, red bubbles mark produce with the shortest seasons, shifting through light to dark green as availability increases, while bubble size still reflects how many months it is in season.

The chart below shows how many products are in season each month. Summer months such as _julio_ (July) with **34** items and _agosto_ (August) with **30** items have the lowest variety, while _octubre_ (October) peaks with **58** items in season, followed by noviembre (November) with **52**. By season, _otoño_ (autumn) has the highest variety, then _invierno_ (winter), followed by _primavera_ (spring). _Verano_ (summer) has the fewest options.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/When-in-season2.png" alt="In what months are different fruits, herbs, and vegetables in season in Spain?" width="800">

</details> <details> <summary>Step 1.2 – What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes?</summary>


**Step overview**

After mapping Spain’s produce seasonality, the next step is to see which fresh ingredients HelloFresh uses most often in its vegetarian recipes. I cleaned and matched ingredient names from the seasonality table with those in the recipes table, ensuring consistent formatting by lowercasing and trimming spaces. Then, I counted how many distinct recipes each ingredient appears in to find the most common ingredients. Finally, I calculated the percentage of total recipes that include each ingredient to show its relative frequency.


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

With the seasonality map complete and the most common ingredients identified, the next step is to measure how closely HelloFresh recipes follow the seasonal calendar. This step calculates the percentage of in-season ingredients in HelloFresh recipes for each month by matching every fresh ingredient with the seasonality table to see if it is in season at that time.

It calculates both:
- the overall monthly percentage of in-season ingredients across all recipes
- the average percentage of in-season ingredients per recipe

The output shows, per month, total fresh ingredient mentions, how many are in season, the overall percentage in season, and the recipe level average percentage in season.

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


<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/recipes-seasonal-each-month.png" width="800">

_Invierno_ (winter) has the lowest share of seasonal ingredients in recipes, with _primavera_ (spring) and _verano_ (summer) only slightly higher. _Atoño_ (autumn) stands out, with 89.51% of recipe ingredients in season. On the monthly level, _Marzo_ (March) is the lowest at 62.92%, followed by _Enero_ (January), _Febrero_ (February), and April, all between 69% and 70%. A similar dip appears in summer, with _Julio_ (July) and _Agosto_ (August) both around 71%.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/seasonal vs recipes use.png" width="800">

When we compare the recipes to the actual availability of fresh produce (see graph), we find months where many items are in season but do not appear in the same proportion in in-season recipes. The clearest example is in _**invierno**_ (winter). During this season, the average recipe freshness is 70,53%, yet on average 44 produce items are in season — the second highest count of all seasons. This mismatch displays there is considerable room to include more seasonal ingredients in winter recipes.

</details>
<details>
  <summary>Step 2.2 – What’s Driving the Winter Freshness Gap? </summary

**Step overview**

First, I looked at the whole year to see which ingredients are **overused** or **underused** compared to how often they are actually in season. This uses a _freshness gap_ measure: the difference between an ingredient’s share of recipes and the share of the year it is in season.

**Overused** – Ingredients used frequently despite being in season for only part of the year.
_Calabacín_ (courgette) appears in 48 recipes (20% of the total) but is only fresh for six months. _Tomate_ (tomato) is fresh for seven months yet features heavily in recipes, along with _albahaca_ (basil), _lima_ (lime), and _berenjena_ (aubergine), each used 7–8% of the time but only in season for four to six months.

**Underused despite high availability** – Ingredients that are in season most of the year but rarely used.
While staples like _ajo_ (garlic), _patata_ (potato), and _zanahoria_ (carrot) are fresh year-round and used often, others such as _pack choi_ (fresh for 10 months but in only one recipe) and _rábano_ (radish, fresh for 9 months but also in just one recipe) are barely present.

With the year-round patterns clear, the next step is to focus on _**invierno**_ (winter). Here, the gap is driven by ingredients like calabacín and tomate, which are out of season all winter yet appear regularly in recipes. This pulls the overall winter freshness down to around 70% despite 44 produce items being in season — a pattern we explore in detail in Step 2.3.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/freshness gap.png" width="800">

</details>

<details>
  <summary>Step 2.3 – What’s Driving the Winter Freshness Gap? </summary

**Step overview**
Now let us zoom in on which ingredients drive the **winter gap**. The aim is to find items that are fresh in _invierno_ (winter) but appear less in recipes, and items that are used a lot when they are not fresh. I use a winter specific freshness index that compares each ingredient’s share of winter recipes with its winter availability month by month. I exclude cebolla as a clear outlier (it appears in 50% of all the recipes, and is always in season). This reveals the main overused and underused drivers in winter and helps explain why recipe freshness sits around seventy percent even though about forty four items are in season.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/Winter Gap.png" width="800">

In this bar chart the usage of the ingredients in recipes is displayed. The colours range from green (in season all winter), orange (1 month of winter in season), and green (in season all winter), to zoom in to see what ingredients drive the winter gap.



</details> <details>
  <summary>Step 2.4 – Low hanging fruit </summary
</details>

Taking these insights together, a useful approach is to focus on produce that is already used, so it is familiar to the public and suppliers are available, but could appear in more recipes. This is especially relevant for **winter**, where we saw the lowest average freshness at **70%**. On average, more produce is in season than is actually reflected in the recipes, highlighting an opportunity to make better use of what is available. In total, there are **44** fresh produce items in season during this period.

The reason for the low seasonality in winter is that for the top 10 products (not including onion) calabacin (courgete) is out of season, and tomate (tomato), lime is only 1 month in season in winter, and also albahaca and berenjena are out of season. For hearthy  wintery dishes it would be good to look at vegetables that are in season the whole winter, think of califlor, calabaza, pack choi, rabano, remolacha. These are in the lowest teir of usage but are in season throughout the winter.


## What I learned (and Challenges I faced)

