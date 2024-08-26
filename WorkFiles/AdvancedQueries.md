# Advanced Queries

## Part One: Investigating Sales over Time

You have a meeting coming up with a new member of the finance team. You know they’ll
be interested in Sales and Profits over time. In preparation, you’re going to investigate
factSales and dimDate

**1.1 What granularity is the factSale table?**

The granularity of the factSale table is Products by Order.

**1.2 From the ERD, what type of relationship exists between the factSale and dimDate tables?**

Many facts to one date. 

**1.3 What is the maximum fiscal year in dimDate?**

``` SQL
SELECT MAX([fiscal year])
FROM dimDate
```

Result: <br>
2017

Answer: <br>
The maximum fiscal year in dimDate is 2017.

**1.4 How many fiscal years do we have sales data for?**
# REVIEW
``` SQL
SELECT
d.[Fiscal Year] As FiscalYear,
SUM(s.[Total Excluding Tax]) AS TotalSalesExcludingTax

FROM factSale AS s
INNER JOIN dimDate AS d
ON s. [Invoice Date Key]=d. [Date]

GROUP BY d. [Fiscal Year]
ORDER BY d. [Fiscal Year] ASC;
```

Result: <br>
| FiscalYear | TotalSalesExcludingTax |
|------------|------------------------|
| 2013       | 38,373,718.70          |
| 2014       | 48,879,106.25          |
| 2015       | 53,827,320.95          |
| 2016       | 31,181,195.30          |

Answer: <br>
We have 4 fiscal years with sales data. 

## Part Two: Sales by Fiscal Period

Your meeting with finance went well. They have a few general questions for you about high
level
performance.

Calculate a report of [Sales Excluding Tax], [Profit], [Quantity Sold]. You may need to analyze data by fiscal
year or month.

**2.1 What were the Sales Excluding tax in fiscal year 2015?**

``` SQL
SELECT
	d. [Fiscal Year] AS FiscalYear,
	SUM(s. [Total Excluding Tax]) AS TotalSalesExcludingTax,
	SUM(s. [Quantity]) AS QuantitySold,
	SUM(s. [Profit]) AS Profit
FROM factSale AS s
	INNER JOIN dimDate AS d
	ON s. [Invoice Date Key]=d. [Date]
GROUP BY d. [Fiscal Year]
ORDER BY FiscalYear DESC;
```

Result: <br>
| FiscalYear | TotalSalesExcludingTax | QuantitySold | Profit       |
|------------|------------------------|--------------|--------------|
| 2016       | 31,181,195.30          | 1,686,288    | 15,486,753.70|
| 2015       | 53,827,320.95          | 2,723,924    | 26,815,310.85|
| 2014       | 48,879,106.25          | 2,526,510    | 24,283,242.45|
| 2013       | 38,373,718.70          | 2,013,906    | 19,143,873.90|

Answer: <br>
In 2015, the sales excluding tax was about 54m. 

**2.2 Which fiscal year appears to have the highest profit $?**

Answer: <br>

``` SQL
SELECT
d.[Fiscal Month Label] AS FiscalMonth,
d.[Fiscal Year] AS FiscalYear,
d. [Fiscal Month Number] AS FiscalMonthNumber,
SUM(s. [Total Excluding Tax]) AS TotalSalesExcludingTax,
SUM(s. [Quantity]) AS QuantitySold,
SUM(s. [Profit]) AS Profit

FROM factSale AS s
INNER JOIN dimDate AS d
ON s. [Invoice Date Key]=d. [Date]

GROUP BY d. [Fiscal Year], d. [Fiscal Month Label], d. [Fiscal Month Number]

ORDER BY FiscalYear DESC, FiscalMonthNumber DESC
```

Answer: <br>
2015

**2.3 What explanation can you offer as to why the profit in 2016 is significantly lower than 2015?**

Answer: <br>
Only 7 months of fiscal year 2016 have sales, meaning the year is not yet complete. 

## Part Three: Top Selling Products

Sales & Marketing want to know and understand the most valuable products in the current
fiscal year 2016.

**3.1 What were the total sales excluding tax in fiscal year 2016?**

``` SQL
SELECT SUM(s. [Total Excluding Tax])
FROM factSale AS s
INNER JOIN dimDate AS d
ON s. [Invoice Date Key]=d. [Date]
WHERE d. [Fiscal Year] = 2016
```

Result: <br>
31181195.30

Answer: <br>
The total sales excluding tax in 2016 were 31 181 195.30. 

**3.2 What was the top selling product in fiscal 2016 so far?**

``` SQL
SELECT TOP 10
p. [Stock Item] AS Product,
SUM(s. [Total Excluding Tax]) AS YTDTotalSalesExcludingTax

FROM factSale AS s
INNER JOIN dimStockItem p
ON s. [Stock Item Key]=p. [Stock Item Key]
INNER JOIN dimDate AS d
ON s. [Invoice Date Key]=d. [Date]

WHERE d. [Fiscal Year] = 2016
GROUP BY p. [Stock Item]
ORDER BY YTDTotalSalesExcludingTax DESC;
```

Result: <br>
| Product                                   | YTDTotalSalesExcludingTax |
|-------------------------------------------|---------------------------|
| Air cushion machine (Blue)               | 1,878,111.00             |
| 20 mm Double sided bubble wrap 50m       | 1,257,120.00             |
| 10 mm Anti static bubble wrap (Blue) 50m | 1,205,820.00             |
| 32 mm Double sided bubble wrap 50m       | 1,052,800.00             |
| 10 mm Double sided bubble wrap 50m       | 1,047,900.00             |
| 20 mm Anti static bubble wrap (Blue) 50m | 1,023,060.00             |
| 32 mm Anti static bubble wrap (Blue) 50m | 994,350.00               |
| Void fill 400 L bag (White) 400L         | 549,500.00               |
| 32 mm Anti static bubble wrap (Blue) 20m | 540,480.00               |
| 20 mm Anti static bubble wrap (Blue) 20m | 453,150.00              

Answer: <br>
The top selling product is the blue air cushion machine.

**3.3 What was the top performing product / salesperson combination in fiscal 2016?**

``` SQL
SELECT TOP 10
e. Employee AS SalesPerson,
p. [Stock Item] AS Product,
SUM(s. [Total Excluding Tax]) AS YTDTotalSalesExcludingTax

FROM factSale AS s

INNER JOIN dimStockItem p
ON s. [Stock Item Key]=p. [Stock Item Key]
INNER JOIN dimDate AS d
ON s. [Invoice Date Key]=d. [Date]
INNER JOIN dimEmployee as e
ON s. [Salesperson Key]=e. [Employee Key]

WHERE d. [Fiscal Year] = 2016
GROUP BY p. [Stock Item], e. Employee
ORDER BY YTDTotalSalesExcludingTax DESC;
```

Result: <br>

| SalesPerson       | Product                               | YTDTotalSalesExcludingTax |
|-------------------|---------------------------------------|---------------------------|
| Amy Trefl         | Air cushion machine (Blue)            | 231,678.00               |
| Anthony Grosse    | Air cushion machine (Blue)            | 231,678.00               |
| Sophia Hinton     | Air cushion machine (Blue)            | 222,183.00               |
| Hudson Onslow     | Air cushion machine (Blue)            | 214,587.00               |
| Archer Lamble     | Air cushion machine (Blue)            | 208,890.00               |
| Taj Shand         | Air cushion machine (Blue)            | 203,193.00               |
| Jack Potter       | Air cushion machine (Blue)            | 201,294.00               |
| Archer Lamble     | 10 mm Anti static bubble wrap (Blue)  | 173,250.00               |
| Lily Code         | 20 mm Double sided bubble wrap 50m   | 167,400.00               |
| Hudson Hollinworth| 20 mm Double sided bubble wrap 50m   | 163,080.00               |

Answer: <br>
It was Amy Trefl and Anthony Grosse (tied), Air cushion machine (Blue).

**3.4 What proportion of total fiscal year 2016 sales do these top performances represent?**
**Think about how you might make your query reusuable if the current year changes** 

``` SQL
SELECT TOP 10
    e.Employee AS SalesPerson,
    p.[Stock Item] AS Product,
    SUM(s.[Total Excluding Tax]) AS YTDTotalSalesExcludingTax,
    FORMAT(
        CAST(
            SUM(s.[Total Excluding Tax]) /
            (
                SELECT SUM(s.[Total Excluding Tax])
                FROM factSale AS s
                INNER JOIN dimDate AS d ON s.[Invoice Date Key] = d.[Date]
                WHERE d.[Fiscal Year] = (
                        SELECT MAX([Fiscal Year]) 
                        FROM factSale AS s 
                        INNER JOIN dimDate AS d ON s.[Invoice Date Key] = d.[Date]
                    )
            ) AS DECIMAL(8, 6)
        ), 'P4'
    ) AS PercentOfSalesYTD
FROM 
    factSale AS s
INNER JOIN 
    dimStockItem p ON s.[Stock Item Key] = p.[Stock Item Key]
INNER JOIN 
    dimDate AS d ON s.[Invoice Date Key] = d.[Date]
INNER JOIN 
    dimEmployee AS e ON s.[Salesperson Key] = e.[Employee Key]
WHERE 
    d.[Fiscal Year] = (
            SELECT MAX([Fiscal Year]) 
            FROM factSale AS s 
            INNER JOIN dimDate AS d ON s.[Invoice Date Key] = d.[Date]
        )
GROUP BY 
    p.[Stock Item], e.Employee
ORDER BY 
    YTDTotalSalesExcludingTax DESC

```

Result: <br>
| SalesPerson      | Product                                     | YTDTotalSalesExcludingTax | PercentOfSalesYTD |
|------------------|---------------------------------------------|---------------------------|-------------------|
| Amy Trefl        | Air cushion machine (Blue)                  | 231678.00                 | 0.7430%           |
| Anthony Grosse   | Air cushion machine (Blue)                  | 231678.00                 | 0.7430%           |
| Sophia Hinton    | Air cushion machine (Blue)                  | 222183.00                 | 0.7125%           |
| Hudson Onslow    | Air cushion machine (Blue)                  | 214587.00                 | 0.6881%           |
| Archer Lamble    | Air cushion machine (Blue)                  | 208890.00                 | 0.6699%           |
| Taj Shand        | Air cushion machine (Blue)                  | 203193.00                 | 0.6516%           |
| Jack Potter      | Air cushion machine (Blue)                  | 201294.00                 | 0.6455%           |
| Archer Lamble    | 10 mm Anti static bubble wrap (Blue) 50m   | 173250.00                 | 0.5556%           |
| Lily Code        | 20 mm Double sided bubble wrap 50m          | 167400.00                 | 0.5368%           |
| Hudson Hollinworth| 20 mm Double sided bubble wrap 50m        | 163080.00                 | 0.5230%           |

Answer: <br>
Amy: 0,7430%
Anthony:  0,7430%

## Part Four: Top Selling Chiller Products

The business recently introduced chilled products into it’s product line. Produce a table
showing individual chiller products, with the quantity sold to date.

**4.1 How many chiller products have zero quantity sold to date?**

``` SQL 
SELECT
    p.[Stock Item],
    SUM(Quantity) AS QuantitySold

FROM factSale AS s
    RIGHT JOIN dimStockItem p
    ON s.[Stock Item Key]=p. [Stock Item Key]
WHERE p. [Is Chiller Stock]=1
GROUP BY (p.[Stock Item])
ORDER BY QuantitySold DESC;
```

Result: <br>
| Stock Item                    | QuantitySold |
|-------------------------------|--------------|
| Chocolate beetles 250g       | 20424        |
| White chocolate snow balls 250g | 18408      |
| Novelty chilli chocolates 250g | 17136       |
| White chocolate moon rocks 250g | 16056      |
| Chocolate sharks 250g        | 15840        |
| Chocolate echidnas 250g      | 15504        |
| Chocolate frogs 250g         | 14976        |
| Novelty chilli chocolates 500g | 9012       |
| White chocolate moon rocks 2kg | NULL        |

Answer: <br>
One chiller product has zero quantity sold to date. 