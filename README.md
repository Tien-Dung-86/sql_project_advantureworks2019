**SQL project** <br/>
**Database: advantureworks2019** <br/>
**Table Schema: https://support.google.com/analytics/answer/3437719?hl=en** <br/>

**Query 01: calculate total visit, pageview, transaction for Jan, Feb and March 2017**
<br/>
Expected output example:			
| month | visit | pageviews | transaction |
| :--- | :---: | :---: | :---: |
| 201701 | 64694 | 257708 | 713 |
| 201702 | 62192 | 233373 | 733 |
| 201703 | 69931 | 259522 | 993 |
<br/>

**Query 02: Bounce rate per traffic source in July 2017**
<br/>
Expected output example:			
| source | total_visits | total_no_of_bounces | bounce_rate |
| :--- | :---: | :---: | :---: |
| google | 38400 | 19798 | 51.557 |
| (direct) | 19891 | 8606 | 43.266 |
| youtube.com | 6351 | 4238 | 66.730 |
| analytics.google.com | 1972 | 1064 | 53.955 |
<br/>

**Query 3: Revenue by traffic source by week, by month in June 2017**
<br/>
Expected output example:			
| time_type | time | source | revenue |
| :--- | :---: | :---: | :---: |
| Month | 201706 | (direct) | 97333.6197 |
| Week | 201724 | (direct) | 30908.9099 |
| Week | 201725 | (direct) | 27295.3199 |
| Month | 201706 | google | 18757.1799 |
<br/>

**Query 04: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.**
<br/>
Expected output example:			
| month | avg_pageviews_purchase | avg_pageviews_non_purchase |
| :--- | :---: | :---: |
| 201706 | 94.0205011389521 | 316.865588463416 |
| 201707 | 124.237551867219 | 334.05655979568 | 
<br/>

**Query 05: Average number of transactions per user that made a purchase in July 2017.**
<br/>
Expected output example:			
| month | avg_total_transactions_per_user |
| :--- | :---: |
| 201707 | 4.16390041493776 | 
<br/>

**Query 06: Average amount of money spent per session. Only include purchaser data in July 2017.**
<br/>
Expected output example:			
| month | avg_revenue_by_user_per_visit |
| :--- | :---: |
| 201707 | 43.85 | 
<br/>

**Query 07: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017**
<br/>
Expected output example:			
| other_purchased_products | quantity |
| :--- | :---: |
| Google Sunglasses | 20 | 
| Google Women's Vintage Hero Tee Black | 7 | 
| SPF-15 Slim & Slender Lip Balm | 6 | 
| Google Women's Short Sleeve Hero Tee Red Heather | 4 | 
<br/>

**Query 08: Calculate cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.**
<br/>
Expected output example:			
| month | num_product_view | num_addtocart | num_purchase | add_to_cart_rate | purchase_rate |
| :--- | :---: | :---: | :---: | :---: | :---: |
| 201701 | 25787 | 7342 | 2143 | 28.47 | 8.31 |
| 201702 | 21489 | 7360 | 2060 | 34.25 | 9.59 |
| 201703 | 23549 | 8782 | 2977 | 37.29 | 12.64 |
<br/>
