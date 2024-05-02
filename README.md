# 🍜 Case Study #1: Danny's Diner
Using SQL to discover customer behaviors and their visiting patterns

<img width="449" alt="Screenshot_3" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/ba8e26fe-7f3d-4347-b37b-7a9b230c621e">

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

## Entity Relationship Diagram
<img width="638" alt="Screenshot_4" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/b34f401f-e5e6-4c84-ad7b-433636e985fc">

## Question and Solution
I have write the queries utilizing Google Bigquery. For more details about the results of the queries, please follow the link: https://console.cloud.google.com/bigquery?sq=277003598955:76d34ad508dc46188d48b81e4f65116e

### 1. What is the total amount each customer spent at the restaurant?
<img width="387" alt="Screenshot_5" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/085c0e8a-5f24-4526-a56a-130f1cdb5cd9">
<img width="773" alt="Screenshot_6" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/a5672183-20b8-48e3-a482-7136a192f4c8">

### Steps:
- Use **JOIN** to merge dannys_diner.sales and dannys_diner.menu tables as sales.customer_id and menu.price are from both tables.
- Use **SUM** to calculate the total sales contributed by each customer.
- Group the aggregated results by sales.customer_id.
### Result:
<img width="309" alt="Screenshot_7" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/f326bd09-4208-4112-9a54-d73b63dc1258">

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

### 2. How many days has each customer visited the restaurant?
<img width="371" alt="Screenshot_8" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/a09f0305-c1cd-4d03-aa86-ea0ccfbba4c3">

### Steps:
To determine the unique number of visits for each customer, utilize **COUNT(DISTINCT order_date)**.
It's important to **apply the DISTINCT** keyword while calculating the visit count to avoid duplicate counting of days. For instance, if Customer A visited the restaurant twice on '2021–01–07', counting **without DISTINCT** would result in 2 days instead of the accurate count of 1 day.
### Result:
<img width="352" alt="Screenshot_18" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/8a34e1b0-6c62-4d09-b3d4-52b59b891604">

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

### 3. What was the first item from the menu purchased by each customer?
<img width="394" alt="Screenshot_9" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/6939aed7-a5c7-4339-ac0e-ece3226e91b6">

### Steps:
- Create a Common Table Expression (CTE) named ordered_sales_cte. Within the CTE, create a new column rank and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by customer_id, and the **ORDER BY** clause orders the rows within each partition by order_date.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, which represents the first row within each customer_id partition.
- Use the GROUP BY clause to group the result by customer_id and product_name.

### Result:
<img width="468" alt="Screenshot_19" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/30bfcbb1-9fe2-430a-9d40-19430b44a8b1">

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
<img width="389" alt="Screenshot_10" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/f72de18d-2e84-4ef7-9465-7e440f8396f4">

### Steps:
- Perform a **COUNT** aggregation on the product_id column and **ORDER BY** the result in descending order using most_purchased field.
- Apply the **LIMIT** 1 clause to filter and retrieve the highest number of purchased items.
### Result:
<img width="376" alt="Screenshot_26" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/22eea6cf-c31b-4277-aa5e-05099ac7aebc">

Most purchased item on the menu is ramen which is 8 times. Yummy!

### 5. Which item was the most popular for each customer?
<img width="677" alt="Screenshot_11" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/1892e855-80dd-42d5-8ac7-c6780a89b3b6">

### Steps:
- Create a CTE named fav_item_cte and within the CTE, join the menu table and sales table using the product_id column.
- Group results by sales.customer_id and menu.product_name and calculate the count of menu.product_id occurrences for each group.
- Utilize the **DENSE_RANK()** window function to calculate the ranking of each sales.customer_id partition based on the count of orders **COUNT(sales.customer_id)** in descending order.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the highest order count for each customer.
### Result:
<img width="619" alt="Screenshot_20" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/e81d5ce3-6a17-4f69-8ae1-39cfb082c739">

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu. 

### 6. Which item was purchased first by the customer after they became a member?
<img width="581" alt="Screenshot_12" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/e4904f5d-679c-4847-887f-45be555e6ad9">

### Steps:
- Create a CTE named joined_as_member and within the CTE, select the appropriate columns and calculate the row number using the **ROW_NUMBER()** window function. The **PARTITION BY** clause divides the data by members.customer_id and the **ORDER BY** clause orders the rows within each members.customer_id partition by sales.order_date.
- Join tables dannys_diner.members and dannys_diner.sales on customer_id column. Additionally, apply a condition to only include sales that occurred after the member's join_date (sales.order_date > members.join_date).
- In the outer query, join the joined_as_member CTE with the dannys_diner.menu on the product_id column.
- In the **WHERE** clause, filter to retrieve only the rows where the row_num column equals 1, representing the first row within each customer_id partition.
Order result by customer_id in ascending order.
### Result:
<img width="500" alt="Screenshot_21" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/754d5c37-d252-4121-bd0d-423975aa44db">

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

### 7. Which item was purchased just before the customer became a member?
<img width="598" alt="Screenshot_13" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/edead32a-9828-4e8b-a839-0e0d41b4a9b1">

### Steps:
- Create a CTE called purchased_prior_member.
- In the CTE, select the appropriate columns and calculate the rank using the **ROW_NUMBER()** window function. The rank is determined based on the order dates of the sales in descending order within each customer's group.
- Join dannys_diner.members table with dannys_diner.sales table based on the customer_id column, only including sales that occurred before the customer joined as a member (sales.order_date < members.join_date).
- Join purchased_prior_member CTE with dannys_diner.menu table based on product_id column.
- Filter the result set to include only the rows where the rank is 1, representing the earliest purchase made by each customer before they became a member.
- Sort the result by customer_id in ascending order.
### Result:
<img width="594" alt="Screenshot_22" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/ce9efd17-5f56-499e-92e2-edb9375b7941">

Both customers' last order before becoming members are sushi.

### 8. What is the total items and amount spent for each member before they became a member?
<img width="427" alt="Screenshot_14" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/6a001e72-ebef-4ed6-a401-5422018a4ed6">

### Steps:
- Select the columns sales.customer_id and calculate the count of sales.product_id as total_items for each customer and the sum of menu.price as total_sales. \
- From dannys_diner.sales table, join dannys_diner.members table on customer_id column, ensuring that sales.order_date is earlier than members.join_date (sales.order_date < members.join_date).
- Then, join dannys_diner.menu table to dannys_diner.sales table on product_id column.
- Group the results by sales.customer_id.
- Order the result by sales.customer_id in ascending order.
### Result:
<img width="442" alt="Screenshot_23" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/40bc7220-6475-4a86-b695-9acb2bc48573">

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
<img width="394" alt="Screenshot_15" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/432e4248-b26a-44eb-91b8-781a38939f50">

### Steps:
Let's break down the question to understand the point calculation for each customer's purchases.
- Each $1 spent = 10 points. However, product_id 1 sushi gets 2x points, so each $1 spent = 20 points.
- Here's how the calculation is performed using a conditional CASE statement:
- If product_id = 1, multiply every $1 by 20 points.
- Otherwise, multiply $1 by 10 points.
- Then, calculate the total points for each customer.
### Result:
<img width="319" alt="Screenshot_24" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/4a1c9ca5-48f7-4f39-a0e5-03ae13ef8ac6">

- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
<img width="660" alt="Screenshot_16" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/6c2ca28d-8bdd-4e18-9b4d-35a66bdf5757">
<img width="769" alt="Screenshot_17" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/1888b3f3-75a6-4e12-9dfa-194827926ac3">

### Assumptions:
On Day -X to Day 1 (the day a customer becomes a member), each $1 spent earns 10 points. However, for sushi, each $1 spent earns 20 points.
From Day 1 to Day 7 (the first week of membership), each $1 spent for any items earns 20 points.
From Day 8 to the last day of January 2021, each $1 spent earns 10 points. However, sushi continues to earn double the points at 20 points per $1 spent.
### Steps:
- Create a CTE called dates_cte.
- In dates_cte, calculate the valid_date by adding 6 days to the join_date and determine the last_date of the month by subtracting 1 day from the last day of January 2021.
- From dannys_diner.sales table, join dates_cte on customer_id column, ensuring that the order_date of the sale is after the join_date (dates.join_date <= sales.order_date) and not later than the last_date (sales.order_date <= dates.last_date).
- Then, join dannys_diner.menu table based on the product_id column.
- In the outer query, calculate the points by using a CASE statement to determine the points based on our assumptions above.
- If the product_name is 'sushi', multiply the price by 2 and then by 10. For orders placed between join_date and valid_date, also multiply the price by 2 and then by 10.
For all other products, multiply the price by 10.
- Calculate the sum of points for each customer.
### Result:
<img width="311" alt="Screenshot_25" src="https://github.com/NuongLe123/Python_RFM_analysis/assets/168357450/1da3c17d-d31d-43e7-ad93-4656ab9fab3f">


  

### Result:
- Total points for Customer A is 1,020.
- Total points for Customer B is 320.


