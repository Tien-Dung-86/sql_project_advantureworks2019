<h1>[SQL] Website Performance Analysis</h1>

<h2>I. Introduction</h2>
<p>In this project, I applied advanced SQL techniques, using Google BigQuery to analyze e-commerce data. I evaluated product performance, sales trends, discount strategies, customer retention, and inventory management. These insights supported the Marketing and Sales teams in making strategic, data-driven decisions to improve business outcomes.</p>

<h2>II. Dataset Access</h2>

**Database: advantureworks2019** <br/>
**Table Schema: https://support.google.com/analytics/answer/3437719?hl=en** <br/>

 <p>The e-commerce dataset is stored in a public Google BigQuery dataset. To access the dataset, follow these steps:</p>
 <ul>
   <li>Log in to your Google Cloud Platform account and create a new project.</li>
   <li>Navigate to the BigQuery console and select your newly created project.</li>
   <li>In the navigation panel, select "Add Data" and then "Search a project".</li>
   <li>Enter the project ID "bigquery-public-data.google_analytics_sample.ga_sessions" and click "Enter".</li>
   <li>Click on the "ga_sessions_" table to open it.</li>
 </ul>

<h2>III. Key Focus</h2>

<ul>
  <li><b>Product Performance Analysis: </b> Evaluated subcategory performance through sales metrics and year-over-year growth rates.</li>
  <li><b>Geographic Sales Patterns: </b> Identified top-performing territories by order quantity across multiple years.</li>
  <li><b>Discount Strategy Assessment: </b> Analyzed seasonal discount costs across product subcategories.</li>
  <li><b>Customer Retention Analysis: </b> Calculated retention rates for successfully shipped orders using cohort analysis.</li>
  <li><b>Inventory Management: </b> Examined stock level trends, month-over-month changes, and stock-to-sales ratios.</li>
  <li><b>Order Status Monitoring:</b> Quantified pending orders and their value to assess fulfillment efficiency.</li>
</ul>

<h2>IV. Exploring Dataset</h2>

<b>Query 01: Calculate total visit, pageview, transaction for Jan, Feb and March 2017</b>
<br/>
```
SELECT 
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
GROUP BY month
ORDER BY month
```

| month | visit | pageviews | transaction |
| :--- | :---: | :---: | :---: |
| 201701 | 64694 | 257708 | 713 |
| 201702 | 62192 | 233373 | 733 |
| 201703 | 69931 | 259522 | 993 |
<br/>
<p>Q1 2017 shows consistent website traffic, with March experiencing a notable spike in transactions (993), indicating improved conversion rates or seasonal effects.</p>


<b>Query 02: Bounce rate per traffic source in July 2017</b>
<br/>
```
SELECT 
  trafficSource.source AS source,
  SUM(totals.visits) AS total_visits,
  SUM(totals.bounces) AS total_no_of_bounces,
  ROUND((100.0 * SUM(totals.bounces) / SUM(totals.visits)), 3) AS bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE _TABLE_SUFFIX BETWEEN '20170701' AND '20170731'
GROUP BY source
ORDER BY total_visits DESC
```
  
| source | total_visits | total_no_of_bounces | bounce_rate |
| :--- | :---: | :---: | :---: |
| google | 38400 | 19798 | 51.557 |
| (direct) | 19891 | 8606 | 43.266 |
| youtube.com | 6351 | 4238 | 66.730 |
| analytics.google.com | 1972 | 1064 | 53.955 |
| Partners | 1788 | 936 | 52.349 |
| m.facebook.com | 669 | 430 | 64.275 |
| google.com | 368 | 183 | 49.728 |
| dfa | 302 | 124 | 41.06 |
| sites.google.com | 230 | 97 | 42.174 |
| facebook.com | 191 | 102 | 53.403 |

<p>Google drives the most traffic but has a high bounce rate. YouTube and Facebook have the highest bounce rates, while direct traffic shows better engagement.</p>
<br/>

<b>Query 03: Revenue by traffic source by week, by month in June 2017</b>
<br/>
```
with 
month_data as(
  SELECT
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
),

week_data as(
  SELECT
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    unnest(hits) hits,
    unnest(product) p
  WHERE p.productRevenue is not null
  GROUP BY 1,2,3
  order by revenue DESC
)

select * from month_data
union all
select * from week_data
order by time_type
;
```			
| time_type | time | source | revenue |
| :--- | :---: | :---: | :---: |
| Month | 201706 | (direct) | 97333.6197 |
| Week | 201724 | (direct) | 30908.9099 |
| Week | 201725 | (direct) | 27295.3199 |
| Month | 201706 | google | 18757.1799 |
| Week | 201723 | (direct) | 17325.679919 |
| Week | 201726 | (direct) | 14914.80995 |
| Week | 201724 | google |9217.169976 |
| Month | 201706 | dfa | 8862.229996 |
| Week |201722 | (direct) | 6888.899975|

<p>Direct traffic consistently generates the highest revenue across weeks and months. Google is the second-largest source. June shows strong overall performance, especially for direct traffic.</p>
<br/>

<b>Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.</b>
<br/>
```
WITH
  purchase_pageview AS(
    SELECT 
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
      SUM(totals.pageviews) /COUNT(DISTINCT(fullVisitorId)) AS avg_pageviews_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE (_TABLE_SUFFIX BETWEEN '20170601' AND '20170731') 
          AND totals.transactions >=1
          AND product.productRevenue IS NOT NULL
    GROUP BY month
  ),
  nonpurchase_pageview AS(
    SELECT 
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
      SUM(totals.pageviews) /COUNT(DISTINCT(fullVisitorId)) AS avg_pageviews_non_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE (_TABLE_SUFFIX BETWEEN '20170601' AND '20170731') 
          AND totals.transactions IS NULL
          AND product.productRevenue IS NULL
    GROUP BY month
  )

SELECT 
  purchase.month,
  avg_pageviews_purchase,
  avg_pageviews_non_purchase
FROM purchase_pageview AS purchase
FULL JOIN nonpurchase_pageview AS non_purchase
USING(month)
ORDER BY month
;
```
	
| month | avg_pageviews_purchase | avg_pageviews_non_purchase |
| :--- | :---: | :---: |
| 201706 | 94.0205011389521 | 316.865588463416 |
| 201707 | 124.237551867219 | 334.05655979568 | 

<p>Non-purchasers exhibit significantly higher average pageviews compared to purchasers. Both groups increased pageviews from June to July, with purchasers showing a larger relative increase.</p>
<br/>

<b>Query 05: Average number of transactions per user that made a purchase in July 2017.</b>
<br/>
```
SELECT 
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
  ROUND(SUM(totals.transactions) /COUNT(DISTINCT(fullVisitorId)), 9) AS avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE (_TABLE_SUFFIX BETWEEN '20170701' AND '20170731') 
      AND totals.transactions >= 1
      AND product.productRevenue IS NOT NULL
GROUP BY month
;
```	
| month | avg_total_transactions_per_user |
| :--- | :---: |
| 201707 | 4.16390041493776 | 

<p>In July 2017, users who made purchases completed an average of 4.16 transactions. This suggests moderate repeat buying behavior among customers within the month.</p>
<br/>

<b>Query 06: Average amount of money spent per session. Only include purchaser data in July 2017.</b>
<br/>
```
SELECT 
  FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
  ROUND(((SUM(product.productRevenue) / SUM(totals.visits)) / 1000000), 2) AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
UNNEST (hits) hits,
UNNEST (hits.product) product
WHERE (_TABLE_SUFFIX BETWEEN '20170701' AND '20170731') 
          AND totals.transactions >= 1
          AND product.productRevenue IS NOT NULL
GROUP BY month
;
```

| month | avg_revenue_by_user_per_visit |
| :--- | :---: |
| 201707 | 43.85 | 
<p>In July 2017, purchasing users spent an average of $43.85 per session. This metric indicates the typical transaction value, useful for understanding customer spending patterns and optimizing pricing strategies.</p>
<br/>

<b>Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017</b>
<br/>
```
WITH buyer_list AS (
    SELECT
        distinct fullVisitorId  
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions>=1
    AND product.productRevenue IS NOT NULL
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list using(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue IS NOT NULL
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```

| other_purchased_products | quantity |
| :--- | :---: |
| Google Sunglasses | 20 | 
| Google Women's Vintage Hero Tee Black | 7 | 
| SPF-15 Slim & Slender Lip Balm | 6 | 
| Google Women's Short Sleeve Hero Tee Red Heather | 4 | 
| Google Men's Short Sleeve Badge Tee Charcoal | 3 | 
| YouTube Men's Fleece Hoodie Black | 3 | 
| 22 oz YouTube Bottle Infuser | 2 | 
| Android Men's Vintage Henley | 2 | 
| Red Shine 15 oz Mug | 2 |
| YouTube Twill Cap | 2 |

<p>Customers who bought the YouTube Men's Vintage Henley also favored Google-branded items, especially Sunglasses. There's a strong preference for casual wear and accessories across Google, YouTube, and Android product lines.</p>
<br/>

**Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.**
<br/>
```
WITH
  product_view AS (
    SELECT 
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
      COUNT(product.v2ProductName) AS num_product_view
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE (_TABLE_SUFFIX BETWEEN '20170101' AND '20170331') 
        AND hits.eCommerceAction.action_type = '2'
    GROUP BY month
  ),
   addtocart_view AS (
    SELECT 
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
      COUNT(product.v2ProductName) AS num_addtocart
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE (_TABLE_SUFFIX BETWEEN '20170101' AND '20170331') 
        AND hits.eCommerceAction.action_type = '3'
    GROUP BY month
  ),
   purchase_view AS (
    SELECT 
      FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d' , date)) AS month,
      COUNT(product.v2ProductName) AS num_purchase
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST (hits) hits,
    UNNEST (hits.product) product
    WHERE (_TABLE_SUFFIX BETWEEN '20170101' AND '20170331') 
        AND hits.eCommerceAction.action_type = '6'
        AND product.productRevenue IS NOT NULL
    GROUP BY month
  )

SELECT 
  product_view.month,
  product_view.num_product_view,
  addtocart_view.num_addtocart,
  purchase_view.num_purchase,
  ROUND(100.00 * (addtocart_view.num_addtocart / product_view.num_product_view), 2) AS add_to_cart_rate,
  ROUND(100.00 * (purchase_view.num_purchase / product_view.num_product_view), 2) AS purchase_rate
FROM product_view
LEFT JOIN addtocart_view
USING(month)
LEFT JOIN purchase_view
USING(month)
ORDER BY month
;
```
   
| month | num_product_view | num_addtocart | num_purchase | add_to_cart_rate | purchase_rate |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 201701 | 25787 | 7342 | 2143 | 28.47 | 8.31 |
| 201702 | 21489 | 7360 | 2060 | 34.25 | 9.59 |
| 201703 | 23549 | 8782 | 2977 | 37.29 | 12.64 |

<p>Product view to purchase conversion rates improved consistently from January to March 2017. March saw the highest engagement, with 37.29% add-to-cart rate and 12.64% purchase rate. This trend indicates increasing effectiveness in converting browsers to buyers, possibly due to improved marketing or user experience.</p>
<br/>

<h2>V. Conclusion</h2>
<p>Overall, this project utilized Google BigQuery to analyze an e-commerce dataset, extracting valuable insights across product performance, customer behavior, and sales patterns. Key findings include improving conversion rates, cross-selling opportunities among branded items, and geographic sales trends. The analysis revealed areas for optimization in marketing strategies, inventory management, and customer experience. By leveraging big data analytics, the project provides a foundation for data-driven decision-making, enabling the business to enhance product offerings, refine pricing strategies, and improve customer retention. These insights position the company to make informed strategic choices, potentially leading to improved performance and growth in the competitive e-commerce landscape.</p>
