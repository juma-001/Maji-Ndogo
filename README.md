# SQL Project
## A background of Maji Ndogo

Maji Ndogo is a fictional country grappling with a severe water crisis, serving as the focal point of a comprehensive data analysis project. The initiative aims to uncover the underlying issues affecting water accessibility and quality, leveraging SQL and data science techniques to inform actionable solutions

## Background:

In Maji Ndogo, residents face significant challenges in accessing clean and safe water. Common issues include:
  1. Long Queues at Shared Taps: Many citizens rely on communal water sources, leading to extended waiting times that disrupt daily life.
  2. Contaminated Water Sources: Some wells and rivers, labeled as clean, have been found to contain biological and chemical contaminants, posing health risks
  3. Infrastructure Failures: A considerable portion of the population deals with non-functional home water systems, forcing them to seek alternative sources
  4. Data Integrity Issues: Discrepancies in water quality assessments have been identified, including instances of inflated scores due to corrupt practices.

## Project Objectives:

The project's primary goal is to analyze various datasets to
  - Identify patterns in water collection and usage
  - Assess the condition and distribution of water infrastructure
  - Detect inconsistencies and potential corruption in data reporting.
  - Provide data-driven recommendations to improve water access and quality
    
## Methodology:

Utilizing SQL, the project examines interconnected tables containing information on employees, water sources, locations, visits, water quality, and pollution levels.
Through clustering, time-series analysis, and data validation techniques, the analysis seeks to present a clear picture of the water crisis in Maji Ndogo and propose sustainable solutions
This initiative not only enhances technical skills in data analysis but also emphasizes the importance of data integrity and its impact on public health and resource management.

```SQL
-- Step 1: Create a temporary dataset 'incorrect_records' to identify discrepancies between auditor and surveyor scores
WITH incorrect_records AS (
    SELECT
        auditor_report.location_id AS audit_location,         -- Location ID from the auditor's report
        auditor_report.true_water_source_score AS auditor_score, -- Auditor's assessment score
        visits.record_id AS record_id,                        -- Unique visit record identifier
        visits.visit_count AS no_of_visits,                   -- Number of visits made
        employee.employee_name AS emp_names,                  -- Name of the assigned employee
        water_quality.subjective_quality_score AS surveyor_score -- Surveyor's assessment score
    FROM
        auditor_report
        JOIN visits ON auditor_report.location_id = visits.location_id
        JOIN employee ON employee.assigned_employee_id = visits.assigned_employee_id
        JOIN water_quality ON visits.record_id = water_quality.record_id
)

-- Step 2: Create a view 'suspect_list' to count the number of mistakes per employee
-- A mistake is defined as a discrepancy between auditor and surveyor scores with only one visit
CREATE VIEW suspect_list AS
SELECT
    emp_names AS employee_name,           -- Employee's name
    COUNT(emp_names) AS no_of_mistakes    -- Total number of discrepancies (mistakes)
FROM
    incorrect_records
WHERE
    auditor_score != surveyor_score       -- Discrepancy between auditor and surveyor scores
    AND no_of_visits = 1                  -- Only consider records with a single visit
GROUP BY
    emp_names
ORDER BY
    no_of_mistakes DESC;

-- Step 3: Retrieve employees whose number of mistakes exceeds the average
SELECT
    employee_name,
    no_of_mistakes
FROM
    suspect_list
WHERE
    no_of_mistakes > (
        SELECT AVG(no_of_mistakes) FROM suspect_list
    );

-- Step 4: Retrieve detailed records for employees with above-average mistakes
SELECT
    emp_names AS employee_name,
    audit_location AS location_id,
    water_quality.statements
FROM
    incorrect_records
WHERE
    emp_names IN (
        SELECT
            employee_name
        FROM
            suspect_list
        WHERE
            no_of_mistakes > (
                SELECT AVG(no_of_mistakes) FROM suspect_list
            )
    );

-- Step 5: Retrieve records with a specific statement indicating suspicion
SELECT
    emp_names AS employee_name,
    audit_location AS location_id,
    water_quality.statements
FROM
    incorrect_records
WHERE
    emp_names IN (
        SELECT employee_name FROM suspect_list
    )
    AND water_quality.statements = 'Suspicion coloured villagers'' descriptions of an official''s aloof demeanour and apparent laziness. The reference to cash transactions casts doubt on their motives.';
```
## Explanation ##

- Temporary Dataset (incorrect_records): This Common Table Expression (CTE) gathers relevant data by joining multiple tables to identify discrepancies between auditor and surveyor assessments.
- View Creation (suspect_list): A view is created to count the number of discrepancies (mistakes) per employee, focusing on cases with only one visit to ensure data reliability.
- Identifying Above-Average Mistakes: Employees whose number of mistakes exceeds the average are identified for further scrutiny.
- Detailed Records Retrieval: Detailed information, including specific statements, is retrieved for employees with above-average mistakes to understand the context of discrepancies.
- Specific Statement Filtering: Records containing a particular statement indicating suspicion are extracted to investigate potential misconduct.

## Key Findings

1.	Long Queue Times at Shared Taps
- Approximately 43% of the population relies on shared taps, leading to average queue times exceeding 120 minutes. 
2.	Non-Functional Home Water Infrastructure
- While 31% of residents have home water systems, nearly half of these systems are non-functional due to issues like broken pipes and faulty pumps.
3.	Contaminated Water Sources
- Only 28% of wells are clean; many labeled as "clean" were found to be biologically or chemically contaminated. 
4.	Discrepancies in Water Quality Assessments
- Auditor reports and surveyor assessments showed inconsistencies, suggesting potential data manipulation or corruption. 
5.	Rural Areas Disproportionately Affected
- Over 60% of water sources are located in rural areas, which also experience higher rates of infrastructure failure and contamination. 

## Recommendations

1.	Prioritize Repairs of Shared Taps
- Improving the functionality of shared taps can significantly reduce queue times and benefit a large portion of the population. 
2.	Fix Existing Home Infrastructure
- Repairing non-functional home water systems will alleviate pressure on shared resources and improve overall water access. 
3.	Address Water Contamination
- Implement filtration systems for contaminated wells and conduct regular water quality assessments to ensure safety. 
4.	Enhance Data Integrity
- Establish robust auditing mechanisms to detect and prevent discrepancies in water quality reporting. 
5.	Focus on Rural Infrastructure Development
- Allocate resources to improve water infrastructure in rural areas, considering their higher vulnerability to water access issues.


