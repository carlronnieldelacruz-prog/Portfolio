# Portfolio
Data Analytics Portfolio

# [Project #1: Data Cleaning in SQL on World Layoffs Data]

This is a project I did that's part of the online course I took on Data Analytics from AlexTheAnalyst on Youtube

* Data was taken from the online course by AlexTheAnalyst
* Using MySQL, the project utilized joins, aggregations, CTEs, and filtering to clean and standardize the data


SELECT * 
FROM layoffs;

First thing, we want to do is to create a staging table with the raw data, which will be our working document to clean the data
CREATE TABLE layoffs_staging \
LIKE layoffs;

INSERT layoffs_staging
SELECT *
FROM layoffs;
-- Now, we perform the following data cleaning
-- 1. Remove Duplicates, if any
-- 2. Standardize the Data
-- 3. Null or blank values
-- 4. Remove any columns
 
 -- Here, we identify duplicate entries by setting row number for each distinct entry
SELECT *, ROW_NUMBER() OVER(PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

WITH duplicate_cte AS
(SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;

-- Here we create a new table that will remove all duplicate entries in our data
CREATE TABLE `layoffs_staging3` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT *
FROM layoffs_staging3
WHERE row_num > 1;

INSERT INTO layoffs_staging3
SELECT *,
ROW_NUMBER() OVER(
	PARTITION BY company, location, industry,
    total_laid_off, percentage_laid_off, `date`,
    stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;

SET SQL_SAFE_UPDATES = 0;

DELETE FROM layoffs_staging3
WHERE row_num > 1;

SET SQL_SAFE_UPDATES = 1;

-- Here, we verify whether we have successfully deleted all duplicate entries in the data
SELECT *
FROM layoffs_staging3
WHERE company = 'Casper';

-- STANDARDIZING THE DATA
-- First, we clean inconsistencies in the data including excess space and extra characters
SELECT company, TRIM(company)
FROM layoffs_staging3;

UPDATE layoffs_staging3
SET company = TRIM(company);

SELECT DISTINCT industry
FROM layoffs_staging3
ORDER BY 1;

-- From the data, it is noticeable that Crypto under the industry has multiple variations such as 'Crypto Currency' or 'CryptoCurrency'.
-- We want to standardize this data
UPDATE layoffs_staging3
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

-- Now, we check every column for inconsistencies
SELECT DISTINCT country
FROM layoffs_staging3
ORDER BY 1;

SELECT DISTINCT country, TRIM(TRAILING '.' FROM country)
FROM layoffs_staging3
ORDER BY 1;

UPDATE layoffs_staging3
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging3;

UPDATE layoffs_staging3
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y'); 

ALTER TABLE layoffs_staging3
MODIFY COLUMN `date` DATE;

-- Working with NULL and BLANK values
SELECT *
FROM layoffs_staging3
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging3
WHERE industry IS NULL
OR industry = '';

SELECT *
FROM layoffs_staging3
WHERE company = 'Airbnb';

-- Here, we will update every NULL and blank data under the industry column to NULL to make it look cleaner.

SELECT *
FROM layoffs_staging3
WHERE industry IS NULL 
OR industry = '';

UPDATE layoffs_staging3
SET industry = NULL
WHERE industry = '';

SELECT *
FROM layoffs_staging3 st3
JOIN layoffs_staging3 st4
	ON st3.company = st4.company
    AND st3.location = st4.location
WHERE (st3.industry IS NULL OR st3.industry = '') AND st4.industry IS NOT NULL;

UPDATE layoffs_staging3 st3
JOIN layoffs_staging3 st4
	ON st3.company = st4.company
    AND st3.location = st4.location
SET st3.industry = st4.industry
WHERE (st3.industry IS NULL OR st3.industry = '') AND st4.industry IS NOT NULL;

SELECT *
FROM layoffs_staging3
WHERE industry IS NULL;

-- WORK WITH THE DATA TO POPULATE NULL VALUES, IF POSSIBLE

-- Now, we want to remove columns and rows that we can and we want to
DELETE FROM layoffs_staging3
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

-- Lastly, we will drop columns that we won't need
ALTER TABLE layoffs_staging3
DROP COLUMN row_num;
