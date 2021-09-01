# **Data Exploration**
This project I using SQL to answer business questions. Businesses can leverage the power of data to win over various business challenges. 

Performance measurement and target-setting are important to the growth process. It will be much easier to invest and manage for growth if the company understand how to compare actual revenue with budget. The main purpose to find out what's working for business and to identify possible opportunities for future expansion.

# Table of content

**Actual Revenue & Bugdet:**

1. [What is the total Revenue of the company this year?](https://github.com/dieuthihongdieu/Data_Exploration#1--what-is-the-total-revenue-of-the-company-this-year)
2. What is the total Revenue Performance YoY?
3. What is the MoM Revenue Performance?
4.  What is the Total Revenue Vs Target performance for the Year?
5.  What is the Revenue Vs Target performance Per Month?

**Key business drivers**

6.  What is the best performing product in terms of revenue this year?
7. What is the product performance Vs Target for the month?
8.  Which account is performing the best in terms of revenue?
9. Which account is performing the best in terms of revenue vs Target?
10. Which account is performing the worst in terms of meeting targets for the year?
11.  Which opportunity has the highest potential and what are the details?
12.  Which account generates the most revenue per marketing spend for this month?

##  What is the total Revenue of the company this year?

```SQL
SELECT --Month_ID,
Sum(revenue) as Total_Revenue_FY21
FROM [DataExploration].[dbo].[Revenue_Raw_Data]
WHERE Month_ID in (SELECT DISTINCT Month_ID 
				    FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
					WHERE Fiscal_Year = 'fy21')
---GROUP BY Month_ID;

```
## 2.  What is the total Revenue Performance YoY?

```SQL
SELECT a.Total_Revenue_FY21,
       b.Total_Revenue_FY20, 
	   a.Total_Revenue_FY21- b.Total_Revenue_FY20     as Dollar_Dif_YoY,
	   (a.Total_Revenue_FY21 /b.Total_Revenue_FY20 )*100.000000  as Perc_Dif_YoY
FROM 
	(
	--FY21
	SELECT--- Month_ID,
	Sum(revenue) as Total_Revenue_FY21
	FROM [DataExploration].[dbo].[Revenue_Raw_Data]
	WHERE Month_ID in (SELECT DISTINCT Month_ID 
						FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
						WHERE Fiscal_Year = 'fy21')
	---GROUP BY Month_ID;		
	) as a,
	--FY20
	(
	SELECT ---Month_ID,
	Sum(revenue) as Total_Revenue_FY20
	FROM [DataExploration].[dbo].[Revenue_Raw_Data]
	WHERE Month_ID in ( SELECT DISTINCT Month_ID -12 /**Year 2021 have 6 months, Year 2020 has 12 months. Month_ID in 2021 - 12 is the same ID in ones in 2020**/
						FROM [DataExploration].[dbo].[Revenue_Raw_Data]
						WHERE Month_ID in (SELECT DISTINCT Month_ID 
											FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
											WHERE Fiscal_Year = 'fy21'))
	---GROUP BY Month_ID;
	) as b
```
## 3.  What is the MoM Revenue Performance?

```SQL
SELECT a.Total_Revenue_TM,
       b.Total_Revenue_LM,
	   a.Total_Revenue_TM - b.Total_Revenue_LM as MoM_Dollar_Differ,
	   (a.Total_Revenue_TM/ b.Total_Revenue_LM) *100    as MoM_Perc_Differ
FROM 
	(
	---This month
		SELECT --Month_ID,
			   Sum(revenue) as Total_Revenue_TM
		FROM [DataExploration].[dbo].[Revenue_Raw_Data]
		WHERE Month_ID in (SELECT MAX(Month_ID)
						   FROM [DataExploration].[dbo].[Revenue_Raw_Data])
		--GROUP BY Month_ID
	) AS a,

	(
	---Last month
		SELECT --Month_ID,
			   Sum(revenue) as Total_Revenue_LM
		FROM [DataExploration].[dbo].[Revenue_Raw_Data]
		WHERE Month_ID in (SELECT MAX(Month_ID)-1
						   FROM [DataExploration].[dbo].[Revenue_Raw_Data])
		--GROUP BY Month_ID
	) As b

```
## 4.  What is the Total Revenue Vs Target performance for the Year?

```SQL 
Select a.Total_Actual_Revenue_FY21, 
       b.Total_Target_Revenue_FY21,
	   a.Total_Actual_Revenue_FY21- b.Total_Target_Revenue_FY21 as Dollar_dif,
       a.Total_Actual_Revenue_FY21 /b.Total_Target_Revenue_FY21- 1 as Perc_dif
FROM 
(
SELECT --Month_ID,
Sum(revenue) as Total_Actual_Revenue_FY21
FROM [DataExploration].[dbo].[Revenue_Raw_Data]
WHERE Month_ID in (SELECT DISTINCT Month_ID 
				    FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
					WHERE Fiscal_Year = 'fy21')
--GROUP BY Month_ID;
) as a,
(
SELECT --Month_ID,
Sum(Target) as Total_Target_Revenue_FY21
FROM [DataExploration].[dbo].[Targets_Raw_Data]
WHERE Month_ID in (SELECT DISTINCT Month_ID /**Actual Year 2021 have 6 months, Target Year 2021 has 12 months. Month_ID in 2021 in table revenue raw data**/
						FROM [DataExploration].[dbo].[Revenue_Raw_Data]
						WHERE Month_ID in (SELECT DISTINCT Month_ID 
											FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
											WHERE Fiscal_Year = 'fy21'))
---GROUP BY Month_ID;
) as b;
```
## 5.  What is the Revenue Vs Target performance Per Month?

```SQL
Select a.Month_ID,
       c.Fiscal_Month,
       a.Total_Actual_Revenue_FY21, 
       b.Total_Target_Revenue_FY21,
	   a.Total_Actual_Revenue_FY21- b.Total_Target_Revenue_FY21 as Dollar_dif,
       a.Total_Actual_Revenue_FY21 /b.Total_Target_Revenue_FY21- 1 as Perc_dif
FROM (
		SELECT Month_ID,
		Sum(revenue) as Total_Actual_Revenue_FY21
		FROM [DataExploration].[dbo].[Revenue_Raw_Data]
		WHERE Month_ID in (SELECT DISTINCT Month_ID 
							FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
							WHERE Fiscal_Year = 'fy21')
		GROUP BY Month_ID
		) as a
	
LEFT jOIN (
		SELECT Month_ID,
		Sum(Target) as Total_Target_Revenue_FY21
		FROM [DataExploration].[dbo].[Targets_Raw_Data]
		WHERE Month_ID in (SELECT DISTINCT Month_ID /**Actual Year 2021 have 6 months, Target Year 2021 has 12 months. Month_ID in 2021 in table revenue raw data**/
								FROM [DataExploration].[dbo].[Revenue_Raw_Data]
								WHERE Month_ID in (SELECT DISTINCT Month_ID 
													FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
													WHERE Fiscal_Year = 'fy21'))
		GROUP BY Month_ID
		) as b
ON a.Month_ID = b.Month_ID

LEFT JOIN( 
          SELECT DISTINCT Month_ID, Fiscal_Month 
		  FROM [DataExploration].[dbo].[ipi_Calendar_lookup]) as c
ON a.Month_ID = c.Month_ID;
```
## 6. What is the best performing product in terms of revenue this year?
```SQL 
SELECT Product_Category,
       Sum(revenue) as Total_Revenue_FY21
FROM [DataExploration].[dbo].[Revenue_Raw_Data]
WHERE Month_ID in (SELECT DISTINCT Month_ID 
				    FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
					WHERE Fiscal_Year = 'fy21')
GROUP BY Product_Category
ORDER BY revenue;
7 - What is the product performance Vs Target for the month?
SELECT a.Month_ID,
	   a.Product_Category,
	   Total_Actual_Revenue_FY21- Total_Target_Revenue_FY21 as Dollar_Differ,
	   Total_Actual_Revenue_FY21/ Total_Target_Revenue_FY21-1 as Perc_Dif
	  
FROM 
(
	SELECT Month_ID,
		   Product_Category,
		   Sum(revenue) as Total_Actual_Revenue_FY21
	FROM [DataExploration].[dbo].[Revenue_Raw_Data]
	WHERE Month_ID in (SELECT DISTINCT MAX(Month_ID) 
						FROM [DataExploration].[dbo].[Revenue_Raw_Data])
	GROUP BY Product_Category, Month_ID
) as a
LEFT JOIN
(
	SELECT Month_ID,
		   Product_Category,
		   Sum(Target) as Total_Target_Revenue_FY21
	FROM [DataExploration].[dbo].[Targets_Raw_Data]
	WHERE Month_ID in (SELECT DISTINCT MAX(Month_ID) 
						FROM [DataExploration].[dbo].[Revenue_Raw_Data])
	GROUP BY Product_Category, Month_ID
) as b
ON a.Month_ID = b. Month_ID and a.Product_Category = b.Product_Category
```

## 8. Which account is performing the best in terms of revenue?
```sqL
SELECT Distinct a.Account_No,
       [New Account Name],
	   Total_Actual_Revenue_FY21
FROM
	(
		SELECT Account_No,
			   Sum(revenue) as Total_Actual_Revenue_FY21
		FROM [DataExploration].[dbo].[Revenue_Raw_Data]
		WHERE Month_ID in (SELECT DISTINCT Month_ID 
				    FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
					WHERE Fiscal_Year = 'fy21')
		GROUP BY Account_No
		--ORDER BY Sum(revenue) DESC
	) AS a
LEFT JOIN 
	( 
		SELECT * 
		FROM [DataExploration].[dbo].[ipi_account_lookup]
	) AS b

on a.Account_No=b.[ New Account No ] 
ORDER BY Total_Actual_Revenue_FY21 DESC
```
## 9. Which account is performing the best in terms of revenue vs Target?
```SQL
SELECT A.Account_No,
       [New Account Name],
	   A.Total_Actual_Revenue_FY21,
	   A.Total_target_Revenue_FY21,
	   Total_Actual_Revenue_FY21/NULLIF(Total_target_Revenue_FY21,0)-1 AS Rev_Vs_Target
FROM  (
	SELECT ISNULL(a.Account_No,b.Account_No) as Account_No, Total_Actual_Revenue_FY21, Total_target_Revenue_FY21
	FROM (
			SELECT Account_No,
				   Sum(revenue) as Total_Actual_Revenue_FY21
			FROM [DataExploration].[dbo].[Revenue_Raw_Data]
			WHERE Month_ID in (SELECT DISTINCT Month_ID 
						FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
						WHERE Fiscal_Year = 'fy21')
			GROUP BY Account_No
			--ORDER BY Sum(revenue) DESC
		) AS a
	
	FULL JOIN (
			SELECT Account_No,
			Sum(target) as Total_target_Revenue_FY21
			FROM [DataExploration].[dbo].[Targets_Raw_Data]
			WHERE Month_ID in (SELECT DISTINCT Month_ID 
						FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
						WHERE Fiscal_Year = 'fy21')
			GROUP BY Account_No 
			) AS b
	ON a.Account_No =b.Account_No
) AS A

LEFT JOIN 
	( 
		SELECT * 
		FROM [DataExploration].[dbo].[ipi_account_lookup]
	) AS b

on A.Account_No=b.[ New Account No ] 
ORDER BY  Total_Actual_Revenue_FY21/NULLIF(Total_target_Revenue_FY21,0)-1 DESC

```
## 10. Which account is performing the worst in terms of meeting targets for the year?
```SQL
SELECT A.Account_No,
       [New Account Name],
	   A.Total_Actual_Revenue_FY21,
	   A.Total_target_Revenue_FY21,
	   ISNULL(Total_Actual_Revenue_FY21,0)/NULLIF(Total_target_Revenue_FY21,0)-1 AS Rev_Vs_Target
FROM  (
	SELECT ISNULL(a.Account_No,b.Account_No) as Account_No, Total_Actual_Revenue_FY21, Total_target_Revenue_FY21
	FROM (
			SELECT Account_No,
				   Sum(revenue) as Total_Actual_Revenue_FY21
			FROM [DataExploration].[dbo].[Revenue_Raw_Data]
			WHERE Month_ID in (SELECT DISTINCT Month_ID 
						FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
						WHERE Fiscal_Year = 'fy21')
			GROUP BY Account_No
			--ORDER BY Sum(revenue) DESC
		) AS a
	
	FULL JOIN (
			SELECT Account_No,
			Sum(target) as Total_target_Revenue_FY21
			FROM [DataExploration].[dbo].[Targets_Raw_Data]
			WHERE Month_ID in (SELECT DISTINCT Month_ID 
						FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
						WHERE Fiscal_Year = 'fy21')
			GROUP BY Account_No 
			) AS b
	ON a.Account_No =b.Account_No
) AS A

LEFT JOIN 
	( 
		SELECT * 
		FROM [DataExploration].[dbo].[ipi_account_lookup]
	) AS b

on A.Account_No=b.[ New Account No ] 
ORDER BY  Total_Actual_Revenue_FY21/NULLIF(Total_target_Revenue_FY21,0)-1

```
## 11. Which opportunity has the highest potential and what are the details?
```SQL
SELECT *FROM [DataExploration].[dbo].[ipi_Opportunities_Data]
WHERE Est_Completion_Month_ID IN (
                                  SELECT DISTINCT Month_ID
								  FROM [DataExploration].[dbo].[ipi_Calendar_lookup]
								  WHERE Fiscal_Year ='fy21')
ORDER BY Est_Opportunity_Value DESC;

```
## 12. Which account generates the most revenue per marketing spend for this month?
```SQL
SELECT 
  ISNULL(a.Account_No, b.Account_No) as Account_No, 
  Total_Actual_Revenue, 
  Marketing_Spend, 
  ISNULL(Total_Actual_Revenue, 0)/ NULLIF(ISNULL(Marketing_Spend, 0),  0) AS Rev_Per_Spend 
From 
  (
    SELECT 
      Account_No, 
      Sum(revenue) as Total_Actual_Revenue 
    FROM 
      [DataExploration].[dbo].[Revenue_Raw_Data] 
    WHERE 
      Month_ID in (
        SELECT DISTINCT Month_ID 
        FROM 
          [DataExploration].[dbo].[ipi_Calendar_lookup] 
        WHERE 
          Fiscal_Year = 'fy21'
      ) 
    GROUP BY  Account_No
  ) AS a 
FULL JOIN (
    SELECT 
      Account_No, 
      SUM (Marketing_Spend) as Marketing_Spend 
    FROM 
      [DataExploration].[dbo].[Marketing_Raw_Data] 
    WHERE 
      Month_ID in (
        SELECT DISTINCT Month_ID 
        FROM 
          [DataExploration].[dbo].[ipi_Calendar_lookup] 
        WHERE 
          Fiscal_Year = 'fy21'
      ) 
    GROUP BY  Account_No
  ) AS b 
ON a.Account_No = b.Account_No 
ORDER BY Rev_Per_Spend DESC;
```





