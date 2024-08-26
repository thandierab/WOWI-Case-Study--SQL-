WOW SQL Casestudy (BIDA)
# Basic Queries

## PART ONE: Explore the database

**1.1 How many rows are there in the FactSale Table?**
``` SQL
SELECT count(*) FROM factSale;
```

Result: <br>
228265

Answer: <br>
There are 228 265 rows in the FactSale Table. 

**1.2 What data type is the Quantity column in FactSale?**

``` SQL
SELECT COLUMN_NAME, DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'factSale' AND COLUMN_NAME = 'quantity';
```

Result: <br>

| Column   | Data_Type |
|----------|-----------|
| Quantity | int       |

Answer:<br>
The Quantity column is an integer data type. i.e it is a whole number.


**1.3 How many columns in the FactOrder table can contain null values?**

``` SQL
SELECT COUNT(COLUMN_NAME) AS Nullable_Columns_Count
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'FactOrder'
AND IS_NULLABLE = 'YES';
```

Result <br>

| Nullable_Columns_Count|
|-----------------------|
| 3                     |

Answer: <br>
Three columns in the FactOrder table can contain null values.

**1.4 Does the Staging_factSale table contain any data?**

``` SQL 
SELECT *
FROM Staging_factSale;
```

Result: <br>
The result is a table with multiple columns, but no data. 

Answer: <br>
No. The table does not contain any data. 

**1.5 Do the collation properties of the server suggest the database is case sensitive?**

``` SQL
SELECT CONVERT (varchar(256), SERVERPROPERTY('collation')) AS collation;
```

Result: <br>

| collation                    |
|------------------------------|
| SQL_Latin1_General_CP1_CI_AS |


Answer: <br>
No. The database is Case Insensetive. 

**1.6 What does the lineage column mean in the fact and dimension tables?**

Answer: <br>
The lineage table combined with the lineage columns helps us understand which data was updated and when.

**1.7 What key is enforced between FactSale and dimCustomer dimension?**

Answer: <br>
In Azure Data Studio. On the left side, I collapsed FactSale table then the Key folder, then I confirmed the key enforced with dimCustomer is the customer key. 
Inspected the ERD and confirmed the Customer_key column is a foreign key in FactSale and a primary key in dimCustomer. 

______________________________________________________________________________________________________________________________

## PART TWO: Basic Customer data

**2.1 How would you best describe the type of customers that we have?**

``` SQL
SELECT DISTINCT [Buying Group], [Category]
FROM dimCustomer;
```

Result: <br>

| Buying Group  | Category     |
|---------------|--------------|
| N/A           | N/A          |
| Tailspin Toys | Novelty Shop |
| WingTip Toys  | Novelty Shop |

Answer:<br>
I would describe our customers as Novelty Shops. 

**2.2 How many rows does the dimCustomer table have?**

``` SQL
SELECT COUNT(*)
FROM dimCustomer;
```

Result: <br>
403

Answer: <br>
The dimCustomer table has 403 rows. 

**2.3 Does the customer dimension include an entry for Customer Unknown?**

``` SQL
SELECT Customer, [Customer Key]
FROM dimCustomer
where customer = 'unknown
```

Result:<br>
| Customer  | Customer Key |
|-----------|--------------|
| Unknown   | 0            |


Answer:<br>
Yes. The dimCustomer table includes an entry for customer unknown. 

**2.4 How many known customers do we have?**

``` SQL
SELECT count(*)
FROM dimCustomer
WHERE customer <> 'unknown';
```

Result: <br>
402

Answer: <br>
There are 402 known customers. 

______________________________________________________________________________________________________________________________
## PART THREE: Stock Item (Product) Questions

**3.1 How many foreign keys does the DimStockItem table have?**

Navigated Azure Studio. On the left pane, i collapsed the Keys folder and noted there is only a primary key and no foreign keys.
Therefore there are no foreign keys in this table. 

**3.2 How many rows are in the dimStockItem table?**

``` SQL
SELECT COUNT(*)
FROM DimStockItem;
```

Result:<br>
229

Answer:<br>
The DimStockItem table has 229 rows. 

**3.3 How many known products are we dealing with?**

``` SQL
SELECT COUNT(*)
FROM dimStockItem
WHERE [Stock Item] <> 'unknown'
```

Result: <br>
228

Answer: <br>
There are 228 known products.

**3.4 What does the “Is Chiller Stock” column mean?**

Answer:<br>
This column confirms wheather the stock item needs to be constantly refrigerated. 1 = True. 0 =False.
______________________________________________________________________________________________________________________________
## PART FOUR: Marketing Questions I

The marketing team are running a new Halloween campaign to reward people who share content on social media. We plan to ship a low cost item to them for free.

**4.1 What is the unit price of the cheapest product that we currently sell (excluding Unknown Items)?**

``` SQL
SELECT TOP 1 [Stock Item], [Unit Price]
FROM dimStockItem
WHERE [Stock Item] <> 'unknown'
ORDER BY [Unit Price] ASC;
```

Result:<br>
| Stock Item                                | Unit Price |
|-------------------------------------------|------------|
|3 kg Courier post bag (White) 300x190x95mm | 0.66       |

Answer:<br>
The unit price of the cheapest item we sell is R0.66. 

**4.2 What is the name of the cheapest product that we currently sell?**

Answer:<br>
Per the above, the cheapest product we sell is the 3 kg Courier post bag (White) 300x190x95mm.

Notice that many of the cheapest items relate to packaging materials. Exclude all items that contain the word box, bag, or carton in their names.

**4.3 What is the cheapest non packaging related product?**

``` SQL
SELECT TOP 1 [Stock Item], [Unit Price]
FROM dimStockItem
WHERE [Stock Item Key] <> 0
  AND [Stock Item] NOT LIKE '%box%'
  AND [Stock Item] NOT LIKE '%BAG%'
  AND [Stock Item] NOT LIKE '%CARTON%'
ORDER BY [Unit Price] ASC;
```

Result:<br>
| Stock Item                                        |Unit Price |
|---------------------------------------------------|------------|
|Packing knife with metal insert blade (Yellow) 9mm	|1.89        |

Answer:<br>
The cheapest non packaging related product is the packing knife. 
______________________________________________________________________________________________________________________________
## PART FIVE: Marketing Questions II

The marketing team have got back to you:
-Since the winner of the social media campaign might be a child, they don’t want to send a knife.
-Instead they have suggested that we send a mug or shirt to the chosen winner.
-The product should also be black if possible.

**5.1 Find a list of products that contain mug or shirt in their name. How many are there?**

``` SQL
SELECT COUNT(*) AS TotalCount
FROM (
    SELECT [Stock Item]
    FROM dimStockItem
    WHERE [Stock Item] LIKE '%mug%' OR
          [Stock Item] LIKE '%shirt%'
) AS SubqueryResult;
```

Result: <br>
| TotalCount| 
|-----------|
| 68        | 

Answer: <br>
There are 68 products that contain mug or shirt in their name. 

**5.2 Since it’s Halloween, how many products meet this description, and are also black?**

``` SQL
SELECT COUNT(*) AS TotalCount
FROM (
SELECT [Stock Item] AS ProductName, [Unit Price], Color
FROM dimStockItem
WHERE ([Stock Item] LIKE '%mug%' OR [Stock Item] LIKE '%shirt%') AND Color = 'Black') AS SUBQUERY;
```

Result: <br>
| TotalCount| 
|-----------|
| 34        | 

Answer: <br>
There are 34 products that contain mug or shirt in their name, and are also black.

**5.3 What is the WWI Stock Item ID of the cheapest product meeting the above conditions?** <br>
**If multiple products have the same price, choose the one with the lowest WWI Stock Item ID.**

``` SQL
SELECT [WWI Stock Item ID], [Stock Item] AS ProductName, [Unit Price]
FROM dimStockItem
WHERE ([Stock Item] LIKE '%mug%' OR [Stock Item] LIKE '% shirt%') AND Color = 'Black'
ORDER BY [Unit Price] ASC;
```

Result: <br>
Note: only the first 5 records are displayed: 

| WWI Stock Item ID | Stock Item                                    | Unit Price |
|-------------------|-----------------------------------------------|------------|
| 17                | DBA joke mug - mind if I join you? (Black)    | 13         |
| 19                | DBA joke mug - daaaaaa-ta (Black)             | 13         |
| 21                | DBA joke mug - you might be a DBA if (Black)  | 13         |
| 23                | DBA joke mug - it depends (Black)             | 13         |
| 25                | DBA joke mug - I will get you in order (Black)| 13         |

Answer: <br>
The WWI Stock ID of the cheapest item that meets these items is 17. 

**5.4 What is the markup of WWI Stock Item ID 29? markup = ( retail price unit price ) / unit price**

``` SQL
SELECT [WWI Stock Item ID], 
(([Recommended Retail Price]-[Unit Price])/[Unit Price]) as Markup
FROM dimStockItem
WHERE [WWI Stock Item ID] = 29
```

Result: <br>
| WWI Stock Item ID | Markup                | 
|-------------------|-----------------------|
| 29                | 0.4953846153846153846 |

Answer: <br>
The markup of WWI Stock Item ID 29 is 49,5%
______________________________________________________________________________________________________________________________
## PART SIX: Delivery Efficiency

The delivery team is exploring ideas to improve delivery efficiency. Instead of making
deliveries to individual customer stores, the team wants to group deliveries by postcode
and buying group. Buying groups purchase inventory on behalf of the stores within the group.

**6.1 How many customers are in each buying group?**

``` SQL
SELECT [Buying Group], COUNT ([Buying Group]) as Total
FROM dimCustomer
GROUP BY [Buying Group]
```

Result: <br>
| Buying Group  | Total |
|---------------|-------|
| N/A           | 1     |
| Tailspin Toys | 201   |
| Wingtip Toys	| 201   |

Answer: <br>
There is 1 unallocated customer. The Tailspin Toys and Wingtip Toys groups each have 201 customers. 

We are trialling a new delivery process with Wingtip Toys. We need to identify clusters of shops from this
buying group that are near each other.

**6.2 Do any postcodes have more than 3 Wingtip Toys shops? If so, which postcode?**

```
SELECT [Postal Code]
FROM dimCustomer
WHERE [Buying Group] = 'Wingtip Toys'
GROUP BY [Postal Code]
HAVING COUNT(*) > 3;
```

Result: <br>
| Postal Code | 
|-------------|
| 90683       | 


Answer: <br>
Yes. One postal code has more that 3 Wingtip Toys shops. It is postcode 90683.

**6.3 If a postcode has been identified, which of the following stores should be included in the delivery efficiency trial?**

``` SQL
SELECT [Postal Code], Customer
FROM dimCustomer
WHERE [Buying Group] = 'Wingtip Toys' AND [Postal Code] IN (
                    SELECT [Postal Code]
                    FROM dimCustomer
                    WHERE [Buying Group] = 'Wingtip Toys'
                    GROUP BY [Postal Code]
                    HAVING COUNT ([Customer Key]) > 3)
```

Result: <br>
| Postal Code | Customer                       |
|-------------|--------------------------------|
| 90683       | Wingtip Toys (Asher, OK)       |
| 90683       | Wingtip Toys (Hollandsburg, IN)|
| 90683       | Wingtip Toys (Ovilla, TX)      |
| 90683       | Wingtip Toys (Montoya, NM)     |

Answer: <br>
The stores that should be included in the delivery efficiency trial are Asher, Hollandsburg, Ovilla and Montoya.  
______________________________________________________________________________________________________________________________
### PART SEVEN: Other Questions

You have a little spare time and decide to explore some other dimensions in the warehouse.

**7.1 Employee Dimension. What proportion of our workforce works in sales?**

``` SQL
SELECT
CAST(COUNT (Employee) AS decimal (8,2 )) /
    (SELECT 
        COUNT (Employee) As EmployeeCnt
        FROM dimEmployee
        WHERE Employee != 'Unknown') As SalesPeoplePctOfTotal
FROM dimEmployee
WHERE Employee != 'Unknown'
AND [Is Salesperson] = 1;
```

Result: <br>
| SalesPeoplePctOfTotal |
|-----------------------|
| 0.5263157894736       |

Answer: <br>
52,6% of thr workforce is in sales.

**7.2 City Dimension. Which sales territory has the highest population** 

``` SQL
SELECT 
    [Sales Territory], 
    COUNT([WWI City ID]) AS NumberOfCities,
    SUM([Latest Recorded Population]) AS TotalPopulation,
    MAX([Latest Recorded Population]) AS PopulationInBiggestCity
FROM dimCity
WHERE City != 'Unknown'
GROUP BY [Sales Territory]
ORDER BY TotalPopulation DESC;
```

Result: <br>
| Sales Territory   | Number of Cities | Total Population | Population in Biggest City |
|-------------------|------------------|------------------|----------------------------|
| Southeast         | 9,063            | 45,207,407       | 821,784                    |
| Far West          | 3,594            | 44,320,861       | 3,792,621                  |
| Mideast           | 4,939            | 40,194,022       | 8,175,133                  |
| Great Lakes       | 5,766            | 32,657,376       | 2,695,598                  |
| Southwest         | 4,278            | 29,516,189       | 2,099,451                  |
| Plains            | 6,120            | 15,831,498       | 459,787                    |
| Rocky Mountain    | 2,213            | 8,794,209        | 600,158                    |
| New England       | 1,721            | 8,746,216        | 617,594                    |
| External          | 246              | 2,071,148        | 381,931                    |

Answer: <br>
It is Southeast.

**7.3 City Dimension. How many cities are in the above territory?**

Answer: <br>
There are 9 063 cities in the territory. 

**7.4 City Dimension. What is the approx. population of the biggest city in that territory?**

Answer: <br>
The approximate population of the biggest city is 800k in the territory. 

**7.5 City Dimension. What is the total population across all sales territories?**

``` SQL
SELECT 
    CASE WHEN [Sales Territory] IS NULL THEN 'Total' ELSE [Sales Territory] END AS [Sales Territory],
    COUNT([WWI City ID]) AS NumberOfCities,
    SUM([Latest Recorded Population]) AS TotalPopulation,
    MAX([Latest Recorded Population]) AS PopulationInBiggestCity
FROM dimCity
WHERE City != 'Unknown'
GROUP BY ROLLUP([Sales Territory])
ORDER BY TotalPopulation DESC;
```

Result: <br>
| Sales Territory   | Number of Cities | Total Population | Population in Biggest City |
|-------------------|------------------|------------------|----------------------------|
| Total             | 37940            | 227,338,926      | 8,175,133                  |
| Southeast         | 9063             | 45,207,407       | 821,784                    |
| Far West          | 3594             | 44,320,861       | 3,792,621                  |
| Mideast           | 4939             | 40,194,022       | 8,175,133                  |
| Great Lakes       | 5766             | 32,657,376       | 2,695,598                  |
| Southwest         | 4278             | 29,516,189       | 2,099,451                  |
| Plains            | 6120             | 15,831,498       | 459,787                    |
| Rocky Mountain    | 2213             | 8,794,209        | 600,158                    |
| New England       | 1721             | 8,746,216        | 617,594                    |
| External          | 246              | 2,071,148        | 381,931                    |

Answer: <br>
The total population across all sales territorries in about 227m.
