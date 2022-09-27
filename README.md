# Case Study 7 Balanced Tree

<img src='https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white)'/>

![image](https://github.com/Shailesh-python/Case-Study-7/blob/main/Case%20Study%207.png)

---

There are 4 data tables available to us in `balanced_tree` schema which we can use to run our SQL queries with:

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
