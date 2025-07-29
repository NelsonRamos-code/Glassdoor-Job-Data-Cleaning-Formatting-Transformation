<h1>Glassdoor Job Data: Cleaning, Formatting & Transformation</h1>



<h2>Description</h2>
This project involved cleaning and transforming real-world Data Science job postings scraped from Glassdoor. The raw dataset contained inconsistent formatting, invalid entries, missing values, and unstructured text fields, making it unreliable for analysis. The goal was to clean, standardize, and engineer features to create a structured, analysis-ready dataset.
<br />


<h2>Languages and Utilities Used</h2>

- <b>SQL</b> 

<h2>Environments Used </h2>

- <b>SQL Server</b> 

<h2>Program walk-through:</h2>


```sql
 /* 


Data Cleaning Project
Data Science Job Posting on glassdoor


*/

-- 1. Remove numbers from ''Company_Name'' column

with CTE1 AS(SELECT Company_Name, LEFT(Company_Name, LEN(Company_Name) - 4) AS new_col
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs)

-- Check if there were indeed more than 4 digits
SELECT Company_Name, new_col
FROM CTE1
WHERE new_col LIKE '%[0-9]%'

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET Company_Name =  LEFT(Company_Name, LEN(Company_Name) - 4)

SELECT *
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs

-- 2. Convert ''Rating'' column from integer to one digit-decimal

SELECT Rating, Rating/10.0
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET Rating = Rating/10.0

--3. Handling Unknown and Inconsisten Data (''-1'' AND ''Unknown'' values)

 -- First,profile suspicious Values for each column

 SELECT COUNT(*), Founded
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Founded
ORDER BY COUNT(*) DESC

-- 118 ''-1'' values found

SELECT COUNT(*), Job_Title
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Job_Title
ORDER BY COUNT(*) DESC

-- No inconsistent data found

SELECT COUNT(*), Salary_Estimate
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Salary_Estimate
ORDER BY COUNT(*) DESC

-- No inconsistent data found

SELECT COUNT(*), Job_Description
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Job_Description
ORDER BY COUNT(*) DESC

-- No inconsistent data found

SELECT COUNT(*), Company_Name
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Company_Name
ORDER BY COUNT(*) DESC

-- 2 NULL/empty values found

SELECT COUNT(*), Location
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Location
ORDER BY COUNT(*) DESC

-- No inconsistent data found

SELECT COUNT(*),Headquarters
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Headquarters
ORDER BY COUNT(*) DESC

-- 31 ''-1'' values found in ''Headquarters'' column **

SELECT COUNT(*),Size
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Size
ORDER BY COUNT(*) DESC

-- 27 ''-1'' and 17 ''Unkown'' values found in ''Size'' column **

SELECT COUNT(*), Type_of_ownership
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Type_of_ownership
ORDER BY COUNT(*) DESC

-- 27 ''-1'' values and 4 ''Unknown'' values found

SELECT COUNT(*), Industry
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Industry
ORDER BY COUNT(*) DESC

-- 71 ''-1'' values found

SELECT COUNT(*), Sector
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Sector
ORDER BY COUNT(*) DESC

-- 71 ''-1'' values found

SELECT COUNT(*), Revenue
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Revenue
ORDER BY COUNT(*) DESC

-- 213 ''Unknown/Non-Applicable'' and 27 ''-1'' values found

SELECT COUNT(*), Competitors
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs
GROUP BY Competitors
ORDER BY COUNT(*) DESC

-- 501 ''-1'' values found

-- Once Identified, Generalize all of them with CASE + SUM

SELECT 
		COUNT(*) AS Total_rows,
		SUM(CASE WHEN Founded IN (-1) THEN 1 ELSE 0 END) AS Bad_Founded,
		SUM(CASE WHEN Company_Name IN (' ') THEN 1 ELSE 0 END) AS bad_company_name,
		SUM(CASE WHEN Headquarters = '-1' THEN 1 ELSE 0 END) AS bad_headquarters,
		SUM(CASE WHEN Size IN ('-1','Unknown') THEN 1 ELSE 0 END) AS bad_size,
		SUM(CASE WHEN Type_of_ownership IN ('-1','Unknown') THEN 1 ELSE 0 END) AS bad_ownership,
		SUM(CASE WHEN Industry = '-1' THEN 1 ELSE 0 END) AS bad_industry,
		SUM(CASE WHEN Sector = '-1' THEN 1 ELSE 0 END) AS bad_sector,
		SUM(CASE WHEN Revenue IN ('-1','Unknown/Non-Applicable') THEN 1 ELSE 0 END) AS bad_revenue,
		SUM(CASE WHEN Competitors = '-1' THEN 1 ELSE 0 END) AS bad_competitors
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs

-- Replace all -1 and Unknown values with na.

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET 
    Founded = CASE WHEN Founded = '1'THEN NULL ELSE Founded END,
    Company_Name = CASE WHEN Company_Name = ' ' THEN NULL ELSE Company_Name END,
    Headquarters = CASE WHEN Headquarters = '-1' THEN NULL ELSE Headquarters END,
    Size = CASE WHEN Size IN ('-1', 'Unknown') THEN NULL ELSE Size END,
    Type_of_ownership = CASE WHEN Type_of_ownership IN ('-1', 'Unknown') THEN NULL ELSE Type_of_ownership END,
    Industry = CASE WHEN Industry = '-1' THEN NULL ELSE Industry END,
    Sector = CASE WHEN Sector = '-1' THEN NULL ELSE Sector END,
    Revenue = CASE WHEN Revenue IN ('-1', 'Unknown/Non-Applicable') THEN NULL ELSE Revenue END,
    Competitors = CASE WHEN Competitors = '-1' THEN NULL ELSE Competitors END;

SELECT *
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs

--4. Feature engineering -  create an average, min and max salary from the string type of Salary Estimate column

		-- Remove the ''(Glassdoor est.)'' at the end of each string

SELECT SUBSTRING(Salary_Estimate, 1, CHARINDEX('(', Salary_Estimate) - 1)
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs;

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET Salary_Estimate = SUBSTRING(Salary_Estimate, 1, CHARINDEX('(', Salary_Estimate) - 1);

		-- Create 'min_salary', 'max_salary', 'avg_salary' columns

SELECT 
  Salary_Estimate, 
  CAST(
    SUBSTRING(
      Salary_Estimate, 
      2, 
      CHARINDEX('K', Salary_Estimate) -2
    ) AS NUMERIC
  ) * 1000 AS min_salary 
FROM 
  DataCleaningProject.dbo.Uncleaned_DS_jobs;

SELECT Salary_Estimate,    
    CAST(
        SUBSTRING(
            Salary_Estimate,
            CHARINDEX('-', Salary_Estimate) + 2,
            CHARINDEX('K', Salary_Estimate, CHARINDEX('K', Salary_Estimate) + 1) - 
            (CHARINDEX('-', Salary_Estimate) + 2)
        ) AS NUMERIC
    ) * 1000 AS max_salary
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs;

ALTER TABLE DataCleaningProject.dbo.Uncleaned_DS_jobs
ADD min_salary NUMERIC;

ALTER TABLE DataCleaningProject.dbo.Uncleaned_DS_jobs
ADD max_salary NUMERIC;

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET min_salary = CAST(
    SUBSTRING(
      Salary_Estimate, 
      2, 
      CHARINDEX('K', Salary_Estimate) -2
    ) AS NUMERIC
  ) * 1000 

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET max_salary = CAST(
        SUBSTRING(
            Salary_Estimate,
            CHARINDEX('-', Salary_Estimate) + 2,
            CHARINDEX('K', Salary_Estimate, CHARINDEX('K', Salary_Estimate) + 1) - 
            (CHARINDEX('-', Salary_Estimate) + 2)
        ) AS NUMERIC
    ) * 1000

ALTER TABLE DataCleaningProject.dbo.Uncleaned_DS_jobs
ADD avg_salary NUMERIC;


UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET avg_salary = ROUND((min_salary + max_salary)/2,1);

SELECT Salary_Estimate, min_salary, max_salary, avg_salary
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs;

--5. Feature engineering -  create a ''Company Age'' and ''state'' column

    -- Check ''Founded'' data type = smallint
SELECT 
    COLUMN_NAME, 
    DATA_TYPE, 
    CHARACTER_MAXIMUM_LENGTH,
    NUMERIC_PRECISION, 
    NUMERIC_SCALE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Uncleaned_DS_jobs'
AND TABLE_SCHEMA = 'dbo';

SELECT CAST(Founded AS INT)
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET Founded = CAST(Founded AS INT)

ALTER TABLE DataCleaningProject.dbo.Uncleaned_DS_jobs
ADD Company_Age INT;

SELECT YEAR(GETDATE()) - Founded AS Company_Age
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs;

UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET Company_Age = YEAR(GETDATE()) - Founded 

        -- Create the ''State'' Column from ''Location''

SELECT 
  Location,
  LTRIM(SUBSTRING(Location, CHARINDEX(',', Location) + 1, LEN(Location)))
FROM DataCleaningProject.dbo.Uncleaned_DS_jobs;

ALTER TABLE DataCleaningProject.dbo.Uncleaned_DS_jobs
ADD State_ varchar(30)


UPDATE DataCleaningProject.dbo.Uncleaned_DS_jobs
SET State_ = LTRIM(SUBSTRING(Location, CHARINDEX(',', Location) + 1, LEN(Location))) ;


```


<h2>📊 Results</h2>

- <b> Standardized formatting across all columns</b> 
- <b>Enhanced analysis capability with new salary and age metrics</b>
- <b>Cleaner categorical data by removing placeholder values</b>
- <b>Improved geographic analysis with state-level data</b> 
