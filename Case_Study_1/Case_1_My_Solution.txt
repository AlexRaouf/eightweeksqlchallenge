{\rtf1\ansi\ansicpg1252\cocoartf1561\cocoasubrtf200
{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\margl1440\margr1440\vieww10800\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 /* --------------------\
   Case Study Questions\
   --------------------*/\
-- 1. What is the total amount each customer spent at the restaurant?\
SELECT s.customer_id, SUM(m.price) as total_amount\
FROM dannys_diner.sales s, dannys_diner.menu m\
WHERE s.product_id = m.product_id\
GROUP BY s.customer_id\
ORDER BY 1;\
\
-- 2. How many days has each customer visited the restaurant?\
SELECT s.customer_id, COUNT(DISTINCT s.order_date)\
FROM dannys_diner.sales s\
GROUP BY 1\
ORDER BY 1;\
\
-- 3. What was the first item from the menu purchased by each customer?\
SELECT customer_id, product_name \
FROM \
	(SELECT s.customer_id, m.product_name, RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) as order_rank\
	FROM dannys_diner.sales s, dannys_diner.menu m\
	WHERE s.product_id = m.product_id) new_table\
WHERE order_rank = 1;\
\
\
-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?\
SELECT customer_id, COUNT(*) as purchased_times FROM \
	(SELECT * FROM dannys_diner.sales s\
	INNER JOIN dannys_diner.menu m\
	ON s.product_id = m.product_id) as New_table\
WHERE product_name = 'ramen'\
GROUP BY 1;\
\
\
-- 5. Which item was the most popular for each customer?\
SELECT customer_id, MODE() WITHIN GROUP (ORDER BY product_name) as item\
FROM (SELECT s.customer_id, m.product_name \
	FROM dannys_diner.sales s, dannys_diner.menu m\
      	WHERE s.product_id = m.product_id) as New_table\
GROUP BY customer_id;\
\
-- 6. Which item was purchased first by the customer after they became a member?\
SELECT customer_id, product_name \
FROM \
	(SELECT s.customer_id, m.product_name, RANK() OVER (ORDER BY m.product_name) as order_rank FROM dannys_diner.sales s \
        JOIN dannys_diner.members  me\
		ON s.customer_id = me.customer_id\
        JOIN dannys_diner.menu m\
       	ON s.product_id = m.product_id\
	WHERE me.join_date>=s.order_date) as new_table\
WHERE order_rank = 1;\
\
\
-- 7. Which item was purchased just before the customer became a member?\
SELECT customer_id, product_name\
FROM\
	(SELECT s.customer_id, m.product_name, RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) as order_rank\
     FROM dannys_diner.sales s\
     JOIN dannys_diner.menu m\
     	ON s.product_id = m.product_id\
     JOIN dannys_diner.members me\
     	ON s.customer_id = me.customer_id\
    WHERE s.order_date < me.join_date\
	GROUP BY 1, 2, s.order_date) as new_table\
WHERE order_rank = 1;\
\
        \
\
-- 8. What is the total items and amount spent for each member before they became a member?\
SELECT s.customer_id, COUNT(m.product_name) as total_items, SUM(m.price) as total_spent\
	FROM dannys_diner.sales s\
	JOIN dannys_diner.menu m\
		ON s.product_id = m.product_id\
    JOIN dannys_diner.members me\
    	ON s.customer_id = me.customer_id\
    WHERE s.order_date < me.join_date\
    GROUP BY s.customer_id\
    ORDER BY 1;\
\
\
-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?\
SELECT s.customer_id, SUM(CASE \
					WHEN m.product_name = 'sushi' THEN m.price * 20\
                    ELSE m.price  * 10\
                    END)\
FROM dannys_diner.sales s\
JOIN dannys_diner.menu m\
	ON s.product_id = m.product_id\
GROUP BY 1\
ORDER BY 1;\
\
\
-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?\
SELECT s.customer_id, SUM(CASE \
	WHEN m.product_name = 'sushi' THEN m.price * 20\
    WHEN s.order_date BETWEEN me.join_date AND me.join_date + INTERVAL '1 week' THEN m.price * 20\
    ELSE m.price  * 10\
    END)\
FROM dannys_diner.sales s\
JOIN dannys_diner.menu m\
	ON s.product_id = m.product_id\
JOIN dannys_diner.members me\
	ON s.customer_id = me.customer_id\
GROUP BY 1\
ORDER BY 1;\
}