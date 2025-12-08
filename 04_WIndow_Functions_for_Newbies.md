# Window Function Pane

![NYEDB Image](NYRDB_Banner.png)

[Home](https://joncwmusic.github.io/Data-Blog-Site/)

Okay so today, it’s all about window functions. Some of you learned `SELECT * FROM table` isn’t enough to do effective analytics.
And now, you have to pray chatgpt knows how to translate your desire to bake a moving average into the downstream table so you can create more financially sound projections to “reduce some of the noise”
or whatever the senior analyst said before delegating the assignment to you.

You may want to try some of these queries out on some data so I've provided some examples towards the end and a few you can try with the W3Schools database throughout the post.

## So What Exactly is a Window Function?

Okay so picture this. You’ve just been tasked with finding the average price of a bunch of products by product category.
“Simple Task.” You think to yourself. Just google how to use a GROUP BY and spit out something like this:

```SQL
SELECT CategoryID, AVG(Price) FROM Products GROUP BY CategoryId;
```

<img width="1512" height="494" alt="image" src="https://github.com/user-attachments/assets/92625896-f1c9-4075-b7f7-65fc954c27f7" />

Or if you’re anything like my old manager and write SQL like you're writing an essay: 

`select CategoryID, avg(Price) from Products group by 1`

No matter how you syntactically butcher your queries, the idea is simple.
whatever groups you want to see aggregated data for (max, min, average, sum, and count) you can break down those totals or aggregates by any categorical dimension or several dimensions 

KPIs like sales by date and product category, or cost by marketing campaign can be made simple with a single query.
That’s cool and all, but the goal post has changed now that you’ve shown you’re capable of doing your job.
Here’s the new task. Instead of only the average your boss wants that same average but wants to "...compare it to the original price. 
And you know what, let’s see the differences of each price away from the average for each product category." 
You think “alright I guess I could join the averages to the original table or something” but young padawan data scientist on the brink of layoff, 
there’s a better way. 

Enter the window function.

Window functions are functions that operate over a "window" of rows. Within that window is the dimension you’re drilling down into. You can get total and aggregates by dimension just a like a group by but **WITHOUT REDUCING THE SIZE OF THE TABLE**. Not only that you can also compare values **BETWEEN RECORDS** in your table with functions like lead, first_value, etc. Let's start with an example.


<img width="1509" height="492" alt="image" src="https://github.com/user-attachments/assets/6a6282ba-21f0-4d51-9824-5a3dcf8206ee" />

Notice how in this image we see the product, price, and category. If we wanted to see the average price compared to the actual price for each product by category we would want something like this:

```SQL
SELECT ProductName, CategoryID, Price
	, AVG(Price) OVER (PARTITION BY CategoryID) AS AvgCategoryPrice 
FROM Products;
```

<img width="1504" height="494" alt="image" src="https://github.com/user-attachments/assets/cc50e7d2-4ed1-4ba8-bdaf-6deac0966a7f" />

Notice how this time there isn’t a group by, but if we look at the output, those averages from the first query pop up.
In fact if you order the data by product category you’ll notice the average is repeated within each category. 
This makes it really easy to look at the spread because now you can just work in an extra column for the differences. Neato.

## Other Contexts for Window Functions

Corporate is now ecstatic as your analysis has given them an excuse to raise prices up to average in hopes of generating more revenue per customer.
Unfortunately, the product line has also stopped receiving customers, causing the entire branch responsible for that product line to go under.
While several hundred lives were just uprooted and dismissed as 
“external market conditions and competitor increases in market leading to dissolution of the branch and consolidation of assets and a greater focus on profitable lines of business” on the company’s annual report,
you still have your job.

Lucky you.

Now your manager is looking for you to build upon your query by showing cumulative revenue across all orders instead of daily revenue.
The reason is because cumulative revenue shows a line going up while daily revenue showcases the revenue declines from the branch closure.
Oh, and he wants to see it for product category as well. Good Luck!

So let’s break this down. What we want to see is a running total.
We can think about the window being a bit more dynamic than in the last example.
In this case the running total is a sum over an expanding window that increments by a step with each successive row within each category you’re grouping.
In the next query you'll see total revenue by product category BUT when you add an order by clause within the window function you'll see it generates the running total day over day within each product_category.

Here's the query without the order by clause:

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

<img width="1514" height="506" alt="image" src="https://github.com/user-attachments/assets/8cd67ecd-d8b0-40de-863c-dc704ecb5b21" />

And here it is with the order by on order date:

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

<img width="1516" height="523" alt="image" src="https://github.com/user-attachments/assets/6bc3aa5b-a363-446b-a6cf-e4ff0c0b8032" />

**PRO TIP:** you can use `UNION ALL` to mix totals and category level reporting.
All you have to do is to duplicate the query and union it with the less granular dimension like so:

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

UNION ALL

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

## The Anatomy of a Window Function

So at this point you're probably seeing partitions and order bys and still might be a little confused by the big picture.
Let's generalize a little bit and take a look at the general structure of a window function.
The anatomy of a window function in general is as follows:

```sql
SELECT AGGREGATE_FUNCTION(numerical_column) OVER (PARTITION BY category_columns ORDER BY sequential_column [FRAME CLAUSE])
```

### The AGGREGATE_FUNCTION could be any of the following functions:
- `COUNT`,`SUM`, `MIN`, `MAX`,`AVG` - We should know these.
- `LEAD`, `LAG` - these give the prior or next row within the window and will return NULLs if there is no next or previous record. You can also use these to dig around the HR database and find out who make the next highest and next lowest salary to you.
- `RANK`,`DENSE_RANK` - The difference between `RANK` and `DENSE_RANK` is that dense_rank will not skip ranks if there is a tie between values in a window and row_number. In other words, if two people in a race tie for second place, the fourth person to finish will get 4th with `RANK` and 3rd with `DENSE_RANK`.
- `LAST_VALUE`, `FIRST_VALUE` - These return the last and first values of your context windows. If you look into the HR database again you'll notice your CEO is either first or last in salary depending on the year which country he or she is filing their taxes from.

### PARTITION, ORDER BY, and the FRAME CLAUSE
- `PARTITION BY` is effectively your "group by" so you can differentiate between categories
- `ORDER BY` just determines the order by which you are incrimenting your window (usually you'll see dates here).
- The `FRAME CLAUSE` might be value based or row based. RANGE makes the clause value dependent and only include values within the range while ROWS specifies which rows to include which could be unbounded. Just like the boundless effort you put in day in and day out to chase the bag.

### Examples

```sql
-- Spread by product category (from average)
SELECT product, price, product_category
	, price - AVG(price) OVER (PARTITION BY product_category) AS price_diff
FROM pProducts;

-- Running Total by product category
SELECT date, product_category, revenue
	, SUM(revenue) OVER (PARTITION BY product_category ORDER BY date ASC) AS cume_revenue
FROM DailyRevenueByDimension;

-- 7 day Moving Average of Revenues by product category
SELECT date, product_category, revenue
	, AVG(revenue) OVER (PARTITION BY product_category ORDER BY date ASC ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
FROM DailyRevenueByDimension;
-- Note if you don't have cumulative dates for whatever reason it won't technically be a 7 day moving average but a 7 record rolling average

-- Compare to latest performance
SELECT date, employee, employee_rating
	, CASE WHEN
		employee_rating > LAST_VALUE(employee_rating) OVER (PARTITION BY employee ORDER BY date ASC)
			THEN 1
		ELSE 0
		END AS need_pip_bool
FROM BadEmployees;

```

## Conclusion
As you can see, window functions give your manager all kinds of insights to present to his immediate direct report
, and of course, the key stakeholders invested in the projects in so far as to say they "spear-headed" it on the resume for their next position.
The key take away is: You've become one step closer to being a pain to replace. 

And that'll do.

[Back to Home](https://joncwmusic.github.io/Data-Blog-Site/)

