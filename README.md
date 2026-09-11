# Portfolio
Data Analytics Portfolio

# [Project #1: Data Cleaning in SQL on World Layoffs Data]

This is a project I did that's part of the online course I took on Data Analytics from AlexTheAnalyst on Youtube

* Data was taken from the online course by AlexTheAnalyst
* Using MySQL, the project utilized joins, aggregations, CTEs, and filtering to clean and standardize the data


-- ============================================================
-- DATA CLEANING IN SQL
-- World Layoffs Dataset
-- Tool: MySQL
-- ============================================================

-- ============================================================
-- 1. REVIEW RAW DATA
-- ============================================================

SELECT *
FROM layoffs;


-- ============================================================
-- 2. CREATE STAGING TABLE
-- ============================================================

CREATE TABLE layoffs_staging
LIKE layoffs;

INSERT INTO layoffs_staging
SELECT *
FROM layoffs;


-- ============================================================
-- 3. IDENTIFY DUPLICATES
-- ============================================================

WITH duplicate_cte AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY
                   company,
                   location,
                   industry,
                   total_laid_off,
                   percentage_laid_off,
                   `date`,
                   stage,
                   country,
                   funds_raised_millions
           ) AS row_num
    FROM layoffs_staging
)

SELECT *
FROM duplicate_cte
WHERE row_num > 1;


-- ============================================================
-- 4. CREATE CLEANING TABLE
-- ============================================================

CREATE TABLE layoffs_staging3 (
    company TEXT,
    location TEXT,
    industry TEXT,
    total_laid_off INT DEFAULT NULL,
    percentage_laid_off TEXT,
    `date` TEXT,
    stage TEXT,
    country TEXT,
    funds_raised_millions INT DEFAULT NULL,
    row_num INT
);


-- ============================================================
-- 5. INSERT DATA WITH ROW NUMBERS
-- ============================================================

INSERT INTO layoffs_staging3
SELECT *,
       ROW_NUMBER() OVER (
           PARTITION BY
               company,
               location,
               industry,
               total_laid_off,
               percentage_laid_off,
               `date`,
               stage,
               country,
               funds_raised_millions
       ) AS row_num
FROM layoffs_staging;


-- ============================================================
-- 6. REMOVE DUPLICATES
-- ============================================================

SET SQL_SAFE_UPDATES = 0;

DELETE FROM layoffs_staging3
WHERE row_num > 1;

SET SQL_SAFE_UPDATES = 1;


-- Verify duplicates were removed
SELECT *
FROM layoffs_staging3
WHERE row_num > 1;


-- ============================================================
-- 7. STANDARDIZE COMPANY NAMES
-- ============================================================

UPDATE layoffs_staging3
SET company = TRIM(company);


-- ============================================================
-- 8. STANDARDIZE INDUSTRY
-- ============================================================

SELECT DISTINCT industry
FROM layoffs_staging3
ORDER BY industry;

UPDATE layoffs_staging3
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';


-- ============================================================
-- 9. STANDARDIZE COUNTRY NAMES
-- ============================================================

UPDATE layoffs_staging3
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';


-- ============================================================
-- 10. STANDARDIZE DATE FORMAT
-- ============================================================

SELECT
    `date`,
    STR_TO_DATE(`date`, '%m/%d/%Y') AS formatted_date
FROM layoffs_staging3;

UPDATE layoffs_staging3
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging3
MODIFY COLUMN `date` DATE;


-- ============================================================
-- 11. HANDLE NULL AND BLANK VALUES
-- ============================================================

UPDATE layoffs_staging3
SET industry = NULL
WHERE industry = '';


-- Find records where industry is missing
SELECT *
FROM layoffs_staging3
WHERE industry IS NULL;


-- Populate missing industry values where possible
UPDATE layoffs_staging3 st3
JOIN layoffs_staging3 st4
    ON st3.company = st4.company
    AND st3.location = st4.location
SET st3.industry = st4.industry
WHERE st3.industry IS NULL
  AND st4.industry IS NOT NULL;


-- ============================================================
-- 12. REMOVE RECORDS WITH INSUFFICIENT DATA
-- ============================================================

DELETE FROM layoffs_staging3
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;


-- ============================================================
-- 13. REMOVE TEMPORARY COLUMN
-- ============================================================

ALTER TABLE layoffs_staging3
DROP COLUMN row_num;


-- ============================================================
-- 14. FINAL REVIEW
-- ============================================================

SELECT *
FROM layoffs_staging3;
