# Case Study 7 Balanced Tree

<img src='https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white)'/>

![image](https://github.com/Shailesh-python/Case-Study-7/blob/main/Case%20Study%207.png)

---

There are 4 data tables available to us in `balanced_tree` schema which we can use to run our SQL queries with:

1. `Product Details`
2. `Product Sales`
3. `Product Hierarcy`
4. `Product Price`

### A. High Level Sales Analysis

## [Question #1](#case-study-questions)
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

## [Question #2](#case-study-questions)
> What is the total generated revenue for all products before discounts?
```SQL
SELECT
	SUM(CAST(S.qty AS INT) * CAST(S.price AS INT)) AS total_revenues
FROM balanced_tree.sales S
```
|total_revenues |
|---------------|
|1289453        |

## [Question #3](#case-study-questions)
> What was the total discount amount for all products?
```SQL
SELECT
	SUM((CAST(S.qty AS INT) * CAST(S.price AS INT)) - CAST(S.discount AS INT)) AS total_discount
FROM balanced_tree.sales S
```
|total_discount |
|---------------|
|1106753        |
