# Window Function Pane

![NYEDB Image](NYRDB_Banner.png)

Okay so today, it’s all about window functions. Some of you learned `SELECT * FROM table` isn’t enough to do effective analytics.
And now, you have to pray chatgpt knows how to translate your desire to bake a moving average into the downstream table so you can create more financially sound projections to “reduce some of the noise”
or whatever the senior analyst said before delegating the assignment to you.

You may want to try some of these queries out on some data so I've provided some examples contextuallized

## So What Exactly is a Window Function?

Okay so picture this. You’ve just been tasked with finding the average price of a bunch of products by product category.
“Simple Task.” You think to yourself. Just google how to use a GROUP BY and spit out something like this:

```SQL
SELECT product_category
  , AVG(price) 
FROM product_catalog 
GROUP BY product_category
```

Or if you’re anything like my old manager and write SQL like you're writing an essay:

```SQL
select product_category, AVG(price) from product_catalog group by 1
```

And if you’re following along with the W3Schools Database:

<img width="1512" height="494" alt="image" src="https://github.com/user-attachments/assets/92625896-f1c9-4075-b7f7-65fc954c27f7" />

```SQL
SELECT CategoryID, AVG(Price) FROM Products GROUP BY CategoryId;
```

No matter how you syntactically butcher your queries the idea is simple, 
whatever groups you want to see aggregated data for (max, min, average, sum, and count) you can break down those totals or aggregate by any categorical dimension or dimensions.
KPIs like sales by date and product category, or cost by marketing campaign can be made simple with a single query.
That’s cool and all, but the goal post has changed now that you’ve shown you’re capable of doing your job.
Here’s the new task. Instead of a simple average your boss wants that same average but wants to "...compare it to the original price. 
And you know what, let’s see the differences of each price away from the average for each product category." 
You think “alright I guess I could join the averages to the original table or something” but young padawan data scientist on the brink of layoff, 
there’s a better way. 

Enter the window function.

Window functions are functions that operate over a "window" of rows. Within that window is the dimension you’re drilling down into. I think it’s better to show an example.

<img width="1509" height="492" alt="image" src="https://github.com/user-attachments/assets/6a6282ba-21f0-4d51-9824-5a3dcf8206ee" />
Notice how in this image we see the product price and category. If we wanted to see the average price compared to the actual price for each product by category we would want something like this:

<img width="1504" height="494" alt="image" src="https://github.com/user-attachments/assets/cc50e7d2-4ed1-4ba8-bdaf-6deac0966a7f" />



```SQL
SELECT ProductName, CategoryID, Price
	, AVG(Price) OVER (PARTITION BY CategoryID) AS AvgCategoryPrice 
FROM Products;
```

Notice how this time there isn’t a group by, but if we look at the output, those averages from the other query pop up.
In fact if you order the data by product category you’ll notice the average is repeated within each category. 
This makes it really easy to look at the spread because now you can just work in an extra column for the differences. Neato.

## Other Contexts for Window Functions

Corporate is now ecstatic as your analysis has given them an excuse to raise prices up to average in hopes of generating more revenue per customer.
Unfortunately, the product line has also stopped receiving customers, causing the entire branch responsible for that product line to go under.
While several hundred lives were just uprooted and dismissed as 
“external market conditions and competitor increases in market leading to dissolution of the branch and consolidation of assets and a greater focus on profitable lines of business” on the company’s annual report,
you still you have your job.

Lucky you.

Now your manager is looking for you to build upon your query by showing cumulative revenue across all orders instead of daily revenue.
The reason is because cumulative revenue shows a line going up while daily revenue showcases the revenue declines from the branch closure.
Oh, and he wants to see it for product category as well. Good Luck!

So let’s break this down. What we want to see is a running total.
We can think about the window being a bit more dynamic than in the last example.
In this case the running total is a sum over an expanding window that increments by a step with each successive row within each category you’re grouping.

<img width="1514" height="506" alt="image" src="https://github.com/user-attachments/assets/8cd67ecd-d8b0-40de-863c-dc704ecb5b21" />

```SQL
SELECT CategoryID, OrderRevenue
	, SUM(OrderRevenue) OVER (PARTITION BY CategoryID) AS CumeRevenue
FROM 
-- think of this part as a temp table/view with the relevant data you need to get category totals
(
SELECT o.OrderDate, p.CategoryID, SUM(d.Quantity*p.Price) AS OrderRevenue 
FROM OrderDetails d 
JOIN Products p ON d.productID = p.ProductID
JOIN Orders o ON d.OrderID = o.OrderID
GROUP BY p.CategoryID, o.OrderDate) tempview
ORDER BY CategoryID, OrderDate
```

<img width="1516" height="523" alt="image" src="https://github.com/user-attachments/assets/6bc3aa5b-a363-446b-a6cf-e4ff0c0b8032" />


```sql
SELECT OrderDate, CategoryID, OrderRevenue
	, SUM(OrderRevenue) OVER (PARTITION BY CategoryID ORDER BY OrderDate) AS CumeRevenue
FROM
-- think of this part as a temp table/view with the relevant data you need to get a running total
(
SELECT o.OrderDate, p.CategoryID, SUM(d.Quantity*p.Price) AS OrderRevenue 
FROM OrderDetails d 
JOIN Products p ON d.productID = p.ProductID
JOIN Orders o ON d.OrderID = o.OrderID
GROUP BY p.CategoryID, o.OrderDate
) tempview  --in some sql dialects, nested queries require aliases for the inner query
ORDER BY CategoryID, OrderDate
```

A little trick you can use to mix totals and category level reporting is to duplicate the query and union it with the less granular dimension like so:

```sql
SELECT OrderDate, CategoryID, OrderRevenue
	, SUM(OrderRevenue) OVER (PARTITION BY CategoryID ORDER BY OrderDate) AS CumeRevenue
FROM
-- think of this part as a temp table/view with the relevant data you need to get a running total
(
SELECT o.OrderDate, p.CategoryID, SUM(d.Quantity*p.Price) AS OrderRevenue 
FROM OrderDetails d 
JOIN Products p ON d.productID = p.ProductID
JOIN Orders o ON d.OrderID = o.OrderID
GROUP BY p.CategoryID, o.OrderDate
) tempview  --in some sql dialects, nested queries require aliases for the inner query

UNION

SELECT OrderDate, 99 AS CategoryID, OrderRevenue
	, SUM(OrderRevenue) OVER (ORDER BY OrderDate) AS CumeRevenue
FROM
-- think of this part as a temp table/view with the relevant data you need to get a running total
(
SELECT o.OrderDate, p.CategoryID, SUM(d.Quantity*p.Price) AS OrderRevenue 
FROM OrderDetails d 
JOIN Products p ON d.productID = p.ProductID
JOIN Orders o ON d.OrderID = o.OrderID
GROUP BY p.CategoryID, o.OrderDate
) tempview  --in some sql dialects, nested queries require aliases for the inner query
ORDER BY 2, 1
```




