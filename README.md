# Healthcare Workforce Survey Analysis (SQL Data Audit)

## Project Overview
An end-to-end SQL cleanup and audit of a 120-row healthcare workforce survey covering doctors, nurses, lab staff, and administrators across multiple hospital departments.

The primary goal was to fix survey entries (missing values, typos, unrealistic entries) without deleting too many rows, then run departmental risk queries to measure workload and burnout.


## Data Cleaning & Preprocessing

1. **Standardized Text**: Unified department names (e.g., mapped `ER` and `emergency` to `Emergency`, `icu` to `ICU`).
2. **Mean Imputation via SQL Join**: Instead of replacing `0` or `NULL` shift hours with a generic global average, calculated the baseline mean for each department and updated missing records using a SQL `JOIN`.
3. **Outliers**: Dropped records reporting > 90 shift hours/week (data entry errors).
4. **Final Cohort**: Cleaned dataset contains **119 valid records**.


## Summary Findings

- **Overall Baseline**: Average weekly workload is **56.24 hours**, average sleep is **6.00 hours/night**, and the average burnout score is **6.38 / 10**.
- **Emergency Department**: Highest direct workload. Emergency doctors averaged **61.08 hours/week** with a **6.69 / 10** burnout rating. Emergency nurses averaged **56.56 hours/week** with **6.56 / 10** burnout.
- **ICU Administration**: ICU Administrators recorded the highest overall burnout score at **7.25 / 10** across **56.00 hours/week**, indicating operational friction in administrative roles.



## SQL Code

```sql
USE healthcare_research_master;

-- Standardize names
UPDATE healthcare_survey_master 
SET department = 'Emergency' 
WHERE department IN ('emergency', 'ER');

UPDATE healthcare_survey_master 
SET department = 'ICU' 
WHERE department IN ('icu', 'Intensive Care');

-- Departmental mean imputation for missing/zero hours
UPDATE healthcare_survey_master t1
JOIN (
    SELECT department, ROUND(AVG(shift_hours_per_week), 0) AS avg_dept_hours
    FROM healthcare_survey_master
    WHERE shift_hours_per_week > 0 AND shift_hours_per_week IS NOT NULL
    GROUP BY department
) t2 ON t1.department = t2.department
SET t1.shift_hours_per_week = t2.avg_dept_hours
WHERE t1.shift_hours_per_week = 0 OR t1.shift_hours_per_week IS NULL;

-- Remove unrealistic shift entries
DELETE FROM healthcare_survey_master 
WHERE shift_hours_per_week > 90;

-- Department & Role Aggregation Query
SELECT 
    department,
    profession,
    COUNT(*) AS sample_size,
    ROUND(AVG(shift_hours_per_week), 2) AS avg_weekly_hours,
    ROUND(AVG(burnout_score), 2) AS avg_burnout_score
FROM healthcare_survey_master
GROUP BY department, profession
ORDER BY department, avg_burnout_score DESC;
```
