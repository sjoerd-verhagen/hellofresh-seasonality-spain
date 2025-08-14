# 🌱 Digging into HelloFresh Spains’s Recipe Data for Fresh, Local, Plant-Forward Ideas 🌱

**Author:** Sjoerd Verhagen  
**Live Tableau dashboard:** [View Here](https://public.tableau.com/views/YOUR-DASHBOARD-LINK)
**Skills applied:**  **SQL** (joins, aggregation, filtering, calculated fields), **Python** (data cleaning, regex, web scraping), **Tableau** (trend analysis, data visualisation), Google Sheets (data QA)

---

## 📌 TL;DR 

I analysed **236 vegetarian recipes** from HelloFresh Spain to see how well they align with **local produce seasonality**. Using Greenpeace’s La Guía de las Frutas y Verduras de Temporada _(The Seasonal Fruit and Vegetable Guide)_, I mapped when each ingredient is in season and compared this to recipe usage. 

While **autumn recipes** scored highest (90%) for freshness, **winter averaged just 71%** despite **44 fresh items** being available, the second-highest number of fresh produce after autumn. The gap is driven by **out-of-season staples** like _calabacín_ (courgette) and _tomate_ (tomato), while **in-season winter vegetables** such as _coliflor_ (cauliflower), _calabaza_ (pumpkin), and _remolacha_ (beetroot) are underused. 

Targeting these could expand HelloFresh’s **seasonal, plant-forward winter menu.** These insights could help HelloFresh improve ingredient sourcing, increase menu variety in low-freshness months, and strengthen marketing of seasonal recipes — potentially reducing costs and improving customer satisfaction.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/tldr2" width="600">

---

## 🎯 Background & Project

Barcelona has a growing food-tech scene, and one company I’ve known for years is **HelloFresh**. As a teenager, I learned to cook with their meal kits. At the time, my mother was busy with her studies and my dad isn’t too keen on cooking. So we opted for their subscription, I had first choice out of the three recipes that arrived each week. At the time, I did not think much about my relationship with food, but those meals taught me the basics that still shape how I cook today.

For this project, I wanted to explore how HelloFresh could innovate in line with today’s trends, especially **local sourcing** and **environmental impact**. While they already label some recipes as “seasonal,” I wanted to take it further: What would it look like to analyse their recipe database and explore new possibilities for more local, plant-forward menus?

I found an online HelloFresh recipe database for every country where the company operates. For **HelloFresh Spain**, the vegetarian category alone spanned about **64 pages**, with roughly 12 recipes per page. Many were duplicates, so cleaning would be important. Direct scraping was tricky as the site is designed to sell recipes, not make them easy to download, so I used a hybrid approach:
I used a hybrid approach to collect and prepare the dataset:
- **Scraped 64 pages** of vegetarian recipes (about **768** listings) using a Python script to quickly gather all recipe links by page
- **Extracted recipe details and ingredient lists** with a web scraper
- **Cleaned and deduplicated** the data in Python, reducing it to **236 unique recipes**
- **Standardised over 300 ingredient names** by removing units and quantities and normalising wording
- **Matched recipes** against a seasonality dataset of **74 produce items** for analysis

After combining and cleaning the data, I ended up with **236 unique vegetarian recipes** from HelloFresh Spain ready for analysis.

---

## 🔍 Key Questions

**Part 1: Descriptive Statistics**
- 1.1 Produce Seasonality in Spain
- 1.1 What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes? 

**Part 2: Exploring new possibilities for more local, plant-forward menus**
- 2.1 How seasonal are HelloFresh recipes each month? Tracking trends and spotting low-season months
- 2.2 Freshness Index: Spotting Overused and Underused Ingredients
- 2.3 What’s Driving the Winter Freshness Gap?

---

## 🛠️ Tools Used

1. Open Web Scraper for collecting rental data
2. Python for cleaning and preparing the data
3. Google Sheets – quick quality checks on the scraped data
4.  SQL for queries and data analysis
5. Tableau for creating clear visual insights and dashboard

---

## Part 1 – Descriptive Statistics

</details>
<details>
<summary>🌱 Step 1.1 – Produce Seasonality in Spain </summary>

**Step Overview**

To understand how well HelloFresh recipes align with what is naturally available, the first step is to map out the seasonality of fresh produce in Spain. For this, I worked with _**Greenpeace’s La Guía de las Frutas y Verduras de Temporada**_ [The Seasonal Fruit and Vegetable Guide], which lists the fruits and vegetables that are in season in Spain each month. I converted the PDF into a CSV, with each row showing the product name, the month, and whether it is in season. The dataset covers **74** fresh products in total. 

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/vegs bubbles2.png" width="800">


Out of these, **5** items are available year-round: ajo (garlic), cebolla (onion), patata (potato), plátano (plantain), and zanahoria (carrot). The median availability is **7** months per year, with produce such as tomate (tomato), brócoli (broccoli), and fresas (strawberries) all falling into this middle range. In the chart, red bubbles mark produce with the shortest seasons, shifting through light to dark green as availability increases, while bubble size still reflects how many months it is in season.

The chart below shows how many products are in season each month. Summer months such as _julio_ (July) with **34** items and _agosto_ (August) with **30** items have the lowest variety, while _octubre_ (October) peaks with **58** items in season, followed by noviembre (November) with **52**. By season, _otoño_ (autumn) has the highest variety, then _invierno_ (winter), followed by _primavera_ (spring). _Verano_ (summer) has the fewest options.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/When-in-season2.png" alt="In what months are different fruits, herbs, and vegetables in season in Spain?" width="800">

</details> <details> <summary>🌱 Step 1.2 – What fresh ingredients are most common in HelloFresh Spain’s vegetarian recipes?</summary>


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
  <summary>🌱Step 2.1 - How seasonal are HelloFresh recipes each month? Tracking trends and spotting low-season months </summary


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
  <summary>🌱Step 2.2 – Freshness Index: Spotting Overused and Underused Ingredients </summary

**Step overview**

First, I looked at the whole year to see which ingredients are **overused** or **underused** compared to how often they are actually in season. This uses a _freshness gap_ measure: the difference between an ingredient’s share of recipes and the share of the year it is in season.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/freshness gap.png" width="600">

**Overused** – Ingredients used frequently despite being in season for only part of the year.
_Calabacín_ (courgette) appears in 48 recipes (20% of the total) but is only fresh for six months. _Tomate_ (tomato) is fresh for seven months yet features heavily in recipes, along with _albahaca_ (basil), _lima_ (lime), and _berenjena_ (aubergine), each used 7–8% of the time but only in season for four to six months.

**Underused despite high availability** – Ingredients that are in season most of the year but rarely used.
While staples like _ajo_ (garlic), _patata_ (potato), and _zanahoria_ (carrot) are fresh year-round and used often, others such as _pack choi_ (fresh for 10 months but in only one recipe) and _rábano_ (radish, fresh for 9 months but also in just one recipe) are barely present.

**What I learned**
With the year-round patterns clear, the next step is to focus on _**invierno**_ (winter). Here, the gap is driven by ingredients like _calabacín_ and _tomate_, which are out of season all winter yet appear regularly in recipes. This pulls the overall winter freshness down to 71% despite 44 produce items being in season — a pattern we explore in detail in Step 2.3.

</details>

<details>
  <summary>🌱Step 2.3 – What’s Driving the Winter Freshness Gap? </summary>

**Step overview**

Now we zoom in on which ingredients drive the **winter gap**. The goal is to identify items that are fresh in _invierno_ (winter) but appear less in recipes, as well as those that are used heavily despite being out of season. _Cebolla_ is excluded as a clear outlier: it appears in 50% of all recipes and is always in season. This highlights the main overused and underused drivers in winter, helping explain why recipe freshness averages around 70% despite about 44 items being in season.

<img src="https://github.com/sjoerd-verhagen/hellofresh-seasonality-spain/blob/main/Winter Gap.png" width="800">

In this bar chart the usage of the ingredients in recipes is displayed. The colours range from green (in season all winter), orange (1 month of winter in season), and green (in season all winter), to zoom in to see what ingredients drive the winter gap.

As seen in the year-round view, _calabacín_ (courgette) and _tomate_ (tomato) are the main overused drivers, followed by _lima_ (lime), _albahaca_ (basil), and _berenjena_ (aubergine). At the other end of the chart are the ingredients with untapped potential: _kale_, _rábano_ (radish), _coliflor_ (cauliflower), _calabaza_ (pumpkin), and _remolacha_ (beetroot). All are in season throughout winter and often feature in heartier seasonal dishes, yet currently appear in relatively few recipes.

</details> <details>
  <summary>🌱Step 2.4 – Summary: Low hanging fruit </summary
</details>

**My advice to HelloFresh**

Taking these insights together, the most promising approach is to focus on produce that is already in use, so it is familiar to customers and suppliers, but that could appear more often in recipes. This is especially relevant for winter, when the average freshness drops to **70**%. During this season, there is actually more produce in season than appears in the recipes, with **44** fresh items available.

The lower seasonality score in winter comes from the fact that, among the top 10 products (excluding _cebolla_ (onion)), _calabacín_ (courgette) is out of season, _tomate_ (tomato) and _lima_ (lime) are only in season for one month, and both _albahaca_ (basil) and _berenjena_ (aubergine) are also out of season. For hearty winter dishes, it would be worth exploring vegetables that stay in season throughout winter, such as **_coliflor_** (cauliflower), **_calabaza_** (pumpkin), _**pack choi**_, **_rábano_** (radish), and **_remolacha_** (beetroot). Many of these are currently underused but could bring variety and freshness to the winter menu.

By increasing the presence of these seasonal winter vegetables, HelloFresh could expand its recipe portfolio with dishes that are both marketable as seasonal and better aligned with local availability.

</details>

## 🚧 What I learned (and Challenges I faced)

Cleaning the data was one of the most time-consuming parts of this project. Ingredient names appeared in multiple formats with different types of wording, which had to be standardised before analysis. For example: 
1. Floretes de coliflor → coliflor (cauliflower)
2. Brotes de espinacas → espinaca (spinach)
3. Remolacha cocida → remolacha (beetroot)
4. Maíz dulce → maíz (corn)

The seasonality source, Greenpeace’s produce guide, was another limitation. It does not include all items grown in Spain or used in HelloFresh recipes, such as _chalota_ (shallot), _certain pumpkins_, _mushrooms_ (portobello, champiñón), and _cebollino_ (chives). This meant some seasonal opportunities could not be analysed.

**How I overcame this**
I created a mapping table to standardise ingredient names and merge duplicates, then tested the cleaned data against a subset of recipes to confirm that joins and seasonal matching worked correctly. While exploring the data in Tableau, I noticed additional naming inconsistencies that had not been addressed in the first pass. This insight prompted me to revisit the raw data, refine the mapping table, and re-run the cleaning process before continuing with the analysis. By doing this, I ensured that the results were based on accurate matches rather than rushing to produce numbers. For produce missing from the Greenpeace list, I documented them separately so they could be incorporated in future iterations of the analysis.
