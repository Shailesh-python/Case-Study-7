# Case Study 7 Balanced Tree

The following are my solutions to the Case Study 7 Balanced Tree questions in 
[Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny")

<img src='https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white)'/>

![image](https://github.com/Shailesh-python/Case-Study-7/blob/main/Case%20Study%207.png)

---

There are 4 [data tables](https://github.com/Shailesh-python/Case-Study-7-Balanced-Tree/blob/main/Data%20Sets) available to us in `balanced_tree` schema which we can use to run our SQL queries with:

1. `Product Details`
2. `Product Sales`
3. `Product Hierarcy`
4. `Product Price`

## Part A. High Level Sales Analysis

### [Question #1](#case-study-questions)
> What was the total quantity sold for all products?
```SQL
SELECT
	PD.product_name,
	SUM(S.qty) AS totalproductsold
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details PD
ON S.prod_id = PD.product_id
GROUP BY PD.product_name
ORDER BY PD.product_name
```
![image](https://user-images.githubusercontent.com/81180156/192161046-1526d22d-d102-492b-972c-e1f8c984c4c2.png)

### [Question #2](#case-study-questions)
> What is the total generated revenue for all products before discounts?
```SQL
SELECT
	SUM(CAST(S.qty AS INT) * CAST(S.price AS INT)) AS total_revenues
FROM balanced_tree.sales S
```
|total_revenues |
|---------------|
|1289453        |

### [Question #3](#case-study-questions)
> What was the total discount amount for all products?
```SQL
SELECT
	SUM((CAST(S.qty AS INT) * CAST(S.price AS INT)) - CAST(S.discount AS INT)) AS total_discount
FROM balanced_tree.sales S
```
|total_discount |
|---------------|
|1106753        |

## Part B. Transaction Analysis

### [Question #1](#case-study-questions)
> How many unique transactions were there?
```SQL
SELECT
	COUNT(DISTINCT S.txn_id) as unique_transactions
FROM balanced_tree.sales S
```
|unique_transactions |
|--------------------|
|2500                |

### [Question #2](#case-study-questions)
> What is the average unique products purchased in each transaction?
```SQL
WITH CTE AS
(
SELECT
	DISTINCT
	s.prod_id,
	s.txn_id
FROM balanced_tree.sales S
),CTE_UNIQUE AS
(	SELECT 
		txn_id,COUNT(*) AS unique_products
	FROM CTE
	GROUP BY txn_id
)	
	SELECT 
		AVG(unique_products) as avg_unique_products
	FROM CTE_UNIQUE
```
|avg_unique_products |
|--------------------|
|     6              |

### [Question #3](#case-study-questions)
> What are the 25th, 50th and 75th percentile values for the revenue per transaction?
```sql
WITH CTE AS
(
SELECT
	S.txn_id,
	SUM(CAST(S.qty AS INT) * CAST(S.price AS INT)) AS revenue	
FROM balanced_tree.sales S
GROUP BY S.txn_id
),CTE_RANK AS
(
	SELECT
		CTE.*,
		ROW_NUMBER() OVER(ORDER BY CTE.revenue ASC) AS RN
	FROM CTE
)	
	SELECT 'pct_25' AS percentile, CTE_RANK.revenue FROM CTE_RANK WHERE CTE_RANK.RN = (SELECT COUNT(*) FROM CTE_RANK) * .25 
	UNION
	SELECT 'pct_50' AS percentile, CTE_RANK.revenue FROM CTE_RANK WHERE CTE_RANK.RN = (SELECT COUNT(*) FROM CTE_RANK) * .50 
	UNION
	SELECT 'pct_75' AS percentile, CTE_RANK.revenue FROM CTE_RANK WHERE CTE_RANK.RN = (SELECT COUNT(*) FROM CTE_RANK) * .75 
```
![image](https://user-images.githubusercontent.com/81180156/192163315-a55fb659-3de4-4a9b-b39c-4e6b517d5559.png)


### [Question #4](#case-study-questions)
> What is the average discount value per transaction?
```SQL
SELECT
	AVG(T.avg_discount) * 1.0 AS avgdiscountpertrans
FROM
(
SELECT
	S.txn_id,
	AVG(S.discount) AS avg_discount
FROM balanced_tree.sales S
GROUP BY S.txn_id
) T
```
| avgdiscountpertrans |
|--------------------|
|     12.0           |

### [Question #5](#case-study-questions)
> What is the percentage split of all transactions for members vs non-members?
```SQL
SELECT 
	SUM(CASE WHEN T.member = 1 THEN 1 ELSE 0 END) AS members,
	SUM(CASE WHEN T.member = 1 THEN 1.0 ELSE 0 END)*100/COUNT(*) AS members_perc,
	SUM(CASE WHEN T.member = 0 THEN 1 ELSE 0 END) AS non_members,
	SUM(CASE WHEN T.member = 0 THEN 1.0 ELSE 0 END)*100/COUNT(*) AS nonmembers_perc
FROM(
SELECT
	DISTINCT
	S.txn_id,
	S.member
FROM balanced_tree.sales S
) T
```
![image](https://user-images.githubusercontent.com/81180156/192248747-3d75ce86-8ebc-4f4a-97b8-132d0369b3c1.png)



### [Question #6](#case-study-questions)
> What is the average revenue for member transactions and non-member transactions?
```SQL
SELECT 
	S.member,
	AVG(CAST(S.price AS INT) * CAST(S.qty AS INT)) AS total_revenue
FROM balanced_tree.sales S
GROUP BY S.member
```
| member | total_revenue |
|--------|---------------|
|   0    |84             |
|   1    |85             |


## Part C. Product Analysis

### [Question #1](#case-study-questions)
> What are the top 3 products by total revenue before discount?
```SQL
SELECT 
	TOP 3
	s.prod_id,
	P.product_name,
	SUM(CAST(S.price AS INT) * CAST(S.qty AS INT)) AS total_revenue
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY S.prod_id,P.product_name
ORDER BY total_revenue DESC
```

|product_id|	product_name	         |total_revenue |
|----------|-----------------------------|--------------|
|2a2353	   |Blue Polo Shirt - Mens	 |217683        |
|9ec847	   |Grey Fashion Jacket - Womens |209304        |
|5d267b	   |White Tee Shirt - Mens	 |152000        |


### [Question #2](#case-study-questions)
> What is the total quantity, revenue and discount for each segment?
```SQL
SELECT 
	P.segment_name,
	SUM(S.qty) AS total_quantity,
	SUM(CAST(S.price AS INT) * CAST(S.qty AS INT)) AS total_revenue,
	SUM(CAST(S.discount AS INT)) AS total_discount
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.segment_name
```
![image](https://user-images.githubusercontent.com/81180156/196401791-9c2e856d-a372-47bf-9c81-6d676536982d.png)

### [Question #3](#case-study-questions)
> What is the top selling product for each segment?
```SQL
SELECT 
	P.segment_name,
	P.product_name,
	SUM(S.qty) AS total_quantity
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.segment_name,P.product_name
ORDER BY P.segment_name ASC, total_quantity DESC
```
![image](https://user-images.githubusercontent.com/81180156/192508797-1e50ecd2-6d2e-4780-9055-8e902a502ec7.png)

### [Question #4](#case-study-questions)
> What is the total quantity, revenue and discount for each category?
```SQL
SELECT 
	P.category_name,
	SUM(S.qty) AS total_quantity,
	SUM(CAST(S.price AS INT) * CAST(S.qty AS INT)) AS total_revenue,
	SUM(CAST(S.price AS INT) * CAST(S.qty AS INT) * CAST(S.discount AS INT)) AS total_discount
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.category_name
```
![image](https://user-images.githubusercontent.com/81180156/192510600-6250f0a8-74f1-4e96-98cd-197b02bd2d92.png)

### [Question #5](#case-study-questions)
> What is the top selling product for each category?
```SQL
SELECT
	* 
FROM
(
SELECT
	P.category_name,
	S.prod_id,
	P.product_name,
	SUM(S.qty) AS total_sales,
	DENSE_RANK() OVER (PARTITION BY P.category_name ORDER BY  SUM(S.qty) DESC) AS RN
FROM balanced_tree.sales S
INNER JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.category_name,S.prod_id,P.product_name
) T
WHERE T.RN = 1
```
![image](https://user-images.githubusercontent.com/81180156/192736032-68f57f75-fa3b-4572-a53f-3e07edc86d24.png)

### [Question #6](#case-study-questions)
> What is the percentage split of revenue by product for each segment?

```SQL
;WITH CTE AS
(
SELECT
	P.segment_name,
	S.prod_id,
	P.product_name,
	SUM(CAST(S.qty AS INT) * CAST(S.price AS INT)) AS total_sales
FROM balanced_tree.sales S
INNER JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.segment_name,S.prod_id,P.product_name
),SEGMENT_CTE AS
	(
		SELECT 
			CTE.segment_name,
			SUM(CTE.total_sales) AS segment_sales
		FROM CTE 
		GROUP BY CTE.segment_name
	)
		SELECT 
			*,sales_per = 100.0 * CTE.total_sales / SEGMENT_CTE.segment_sales
		FROM CTE
		LEFT JOIN SEGMENT_CTE 
			ON CTE.segment_name = SEGMENT_CTE.segment_name
		ORDER BY CTE.segment_name ASC
```
![image](https://user-images.githubusercontent.com/81180156/192749680-d4d9e5e1-f5f9-403b-ab32-043a5fb7f10b.png)


### [Question #7](#case-study-questions)
> What is the percentage split of revenue by segment for each category?

```sql
;WITH CTE AS
(
SELECT
	P.category_name,
	P.segment_name,
	SUM(CAST(S.qty AS INT) * CAST(S.price AS INT)) AS total_sales
FROM balanced_tree.sales S
INNER JOIN balanced_tree.product_details P
	ON S.prod_id = P.product_id
GROUP BY P.category_name,P.segment_name
),SEGMENT_CTE AS
	(
		SELECT 
			CTE.category_name,
			SUM(CTE.total_sales) AS category_sales
		FROM CTE 
		GROUP BY CTE.category_name
	)
		SELECT 
			*,sales_per = 100.0 * CTE.total_sales / SEGMENT_CTE.category_sales
		FROM CTE
		LEFT JOIN SEGMENT_CTE 
			ON CTE.category_name = SEGMENT_CTE.category_name
		ORDER BY CTE.category_name ASC
```
![image](https://user-images.githubusercontent.com/81180156/192751523-803a23c6-0988-4ca8-b686-aba06fd10831.png)


### [Question #8](#case-study-questions)
> What is the percentage split of revenue by segment for each category?

```sql
SELECT 
	PD.category_id,
	PD.category_name,
	SUM(CAST(S.price AS INT) * CAST(S.qty AS INT)) AS total_revenue,
	CAST(100.0 * SUM(CAST(S.price AS INT) * CAST(S.qty AS INT))/(SELECT(SUM(CAST(price AS INT) * CAST(qty AS INT))) FROM balanced_tree.sales) AS decimal(5,2))
	 AS split_percent
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details PD
ON S.prod_id = PD.product_id
GROUP BY PD.category_id,PD.category_name
```

![image](https://user-images.githubusercontent.com/81180156/193471666-0f74aa14-9071-47c4-97e2-7e336b36a6a6.png)

### [Question #9](#case-study-questions)
> What is the total transaction “penetration” for each product?

```sql

--Hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions.

SELECT 
	PD.product_id,
	PD.product_name,
	SUM(CASE WHEN S.qty >= 1 THEN 1 ELSE NULL END) AS total_transactions,
	SUM(CASE WHEN S.qty >= 1 THEN 1.0 ELSE NULL END)
		/ (SELECT COUNT(DISTINCT balanced_tree.sales.txn_id)/100 FROM balanced_tree.sales) AS penetration_percentage
FROM balanced_tree.sales S
LEFT JOIN balanced_tree.product_details PD
ON S.prod_id = PD.product_id
GROUP BY PD.product_id,PD.product_name
```
![image](https://user-images.githubusercontent.com/81180156/193472914-ee60b16f-779f-402b-a8d9-aea46f84db58.png)

### [Question #10](#case-study-questions)
> What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?

```sql
;WITH CTE AS
(
SELECT
	T.txn_id
FROM
(
SELECT 
	DISTINCT S.txn_id, S.prod_id 
FROM balanced_tree.sales S
) T
GROUP BY T.txn_id
HAVING COUNT(T.prod_id) = 3
)
	SELECT * 
	FROM balanced_tree.sales S
	LEFT JOIN balanced_tree.product_details PD
	ON S.prod_id = PD.product_id
	WHERE S.txn_id IN (SELECT txn_id FROM CTE)
```
![image](https://user-images.githubusercontent.com/81180156/193474866-e2dde518-b67a-48c9-bb0b-d31536e229f2.png)

### [SUPER BONUS](#case-study-questions)
> What are the quantity, revenue, discount and net revenue from the top 3 products in the transactions where all 3 were purchased?
Answers may vary as per the question interpretation.
```sql
-- Getting top 3 product on the basis of trasactions.
SELECT
	TOP 3
	S.prod_id,
	COUNT(S.prod_id) AS total_transactions,
	SUM(S.qty) AS total_quantity,
	SUM(CAST(S.QTY AS INT) * CAST(S.PRICE AS INT)) AS revenue,
	SUM(CAST(S.discount AS INT)) AS total_discounts,
	SUM(CAST(S.QTY AS INT) * CAST(S.PRICE AS INT)) - SUM(CAST(S.discount AS INT)) AS net_revenue
FROM balanced_tree.sales S
GROUP BY S.prod_id
ORDER BY total_transactions DESC
```

![image](https://user-images.githubusercontent.com/81180156/193475729-c8384e2e-3736-40e7-be19-49e63729ee48.png)












