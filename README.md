# Analysis of Customer Behaviour-  MSSQL-Server
This project and the data used was part of a case study which can be found [here](https://8weeksqlchallenge.com/case-study-1). This exercise focuses on understanding customer behavious by examining patterns, trends, and factors influencing customer spending in order to gain insights into their preferences, purchasing habits, and potential areas for improvement in menu offerings and marketing strategies to deploy for Danny`s Dinner.

### Background
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen. Danny’s Diner is in need of assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favorite. Having this deeper connection with his customers will help him deliver a better and more personalized experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for me to write fully functioning SQL queries to help him answer his questions.

### Entity Relationship Daigram

<img width="529" alt="Screenshot 2024-02-04 004256" src="https://github.com/Dataskillx/Analysis-of-Customer-Behaviour---MSSQL-Server/assets/157766946/63758aba-3293-49cd-bb71-ecbc9abe59d9">

### Skills Applied
- Window Functions
- CTEs
- Aggregations
- JOINs
- Write scripts to generate basic reports that can be run every period

### Questions Explored
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### Some Queries Used
Q6 - Which item was purchased first by the customer after they became a member?

```SQL
WITH first_purchase_after_membership AS (
	SELECT s.customer_id, MIN(s.order_date) AS first_purchase_date
	FROM sales s
	JOIN members mb ON s.customer_id = mb.customer_id
	WHERE s.order_date >= mb.join_date
	GROUP BY s.customer_id
)
SELECT fpam.customer_id, m.product_name
FROM first_purchase_after_membership fpam
JOIN sales s On s.customer_id = fpam.customer_id
AND fpam.first_purchase_date = s.order_date
JOIN menu m ON s.product_id = m.product_id;

```

Q9 - If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT s.customer_id, SUM(
	CASE
		WHEN m.product_name = 'sushi' THEN m.price*20
		ELSE m.price*10 END) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;

```

Q10 - In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
SELECT s.customer_id, SUM(
	CASE
		WHEN s.order_date BETWEEN mb.join_date AND DATEADD(day,7,mb.join_date)
		THEN m.price*20
		WHEN m.product_name ='sushi' THEN m.price*20
		ELSE m.price*10 END) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
LEFT JOIN members mb  ON s.customer_id = mb.customer_id
WHERE s.customer_id  IN ('A', 'B') AND s.order_date <= '2021-01-31'
GROUP BY s.customer_id;

```

### Insights
- Analysis revealed customer B had the most visit 6 times in Jan 2021.
- The favourite dish at Danny’s Diner’s is ramen, followed by curry and sushi.
- Customer A loves ramen, Customer C loves only ramen whereas Customer B seems to enjoy sushi, curry and ramen equally.
- The last item ordered by Customers A and B before they became members are sushi and curry. Does it mean both of these items are the deciding factor?

### Reference
- [Case study](https://8weeksqlchallenge.com/case-study-1/)
- [How to document project on Github by Irene Nafula](https://www.youtube.com/watch?v=0N9xekdKCwk&t=1217s)
