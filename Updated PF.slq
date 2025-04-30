-- Emergency Admissions and Patient Flow Analysis --

-- 1. Total number of unique patients in the system
SELECT COUNT(DISTINCT patientID) AS total_patients
FROM Patient;

-- 2. Number of patients diagnosed with flu
SELECT COUNT(*) AS patients_with_flu
FROM Patient
WHERE flu = 1;

-- 3. Number of encounters by encounter class
SELECT 
    encounterClass, 
    COUNT(*) AS encounter_count
FROM Encounter
GROUP BY encounterClass
ORDER BY encounter_count DESC;

-- 4. Total and average base encounter cost by encounter class
SELECT 
    encounterClass,
    SUM(TRY_CAST(baseEncounterCost AS FLOAT)) AS total_cost,
    AVG(TRY_CAST(baseEncounterCost AS FLOAT)) AS avg_cost
FROM Encounter
GROUP BY encounterClass
ORDER BY total_cost DESC;

-- 5. Number of emergency encounters per month
SELECT 
    FORMAT(start, 'yyyy-MM') AS admission_month,
    COUNT(*) AS emergency_admissions
FROM Encounter
WHERE encounterClass = 'emergency'
GROUP BY FORMAT(start, 'yyyy-MM')
ORDER BY admission_month;

-- 6. Number of encounters by encounter class and type
SELECT
    encounterClass,
    encounterType,
    COUNT(*) AS encounter_count
FROM Encounter
GROUP BY encounterClass, encounterType
ORDER BY encounter_count DESC;

-- 7. Average length of stay (in days) by encounter class
SELECT 
    encounterClass,
    AVG(DATEDIFF(DAY, start, stop)) AS avg_length_of_stay
FROM Encounter
GROUP BY encounterClass
ORDER BY avg_length_of_stay DESC;

-- 8. Top 5 most common encounter reasons
SELECT
    encounterReason,
    COUNT(*) AS reason_count
FROM Encounter
GROUP BY encounterReason
ORDER BY reason_count DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;

-- 9. Total cost of encounters by payer category
SELECT
    payerCategory,
    SUM(TRY_CAST(baseEncounterCost AS FLOAT)) AS total_cost
FROM Encounter
GROUP BY payerCategory
ORDER BY total_cost DESC;

-- 10. Rank encounters by cost within each payer category
SELECT 
    payerCategory,
    encounterID,
    baseEncounterCost,
    RANK() OVER (PARTITION BY payerCategory ORDER BY TRY_CAST(baseEncounterCost AS FLOAT) DESC) AS rank
FROM Encounter
ORDER BY payerCategory, rank;

-- 11. Percentage of emergency admissions by encounter type
WITH EncounterTypeCounts AS (
    SELECT encounterType, COUNT(*) AS encounter_count
    FROM Encounter
    WHERE encounterClass = 'Emergency'
    GROUP BY encounterType
),
TotalEmergencyCount AS (
    SELECT COUNT(*) AS total_emergency_count
    FROM Encounter
    WHERE encounterClass = 'Emergency'
)
SELECT 
    etc.encounterType,
    etc.encounter_count,
    (etc.encounter_count * 100.0 / tec.total_emergency_count) AS percentage_of_total
FROM EncounterTypeCounts etc
CROSS JOIN TotalEmergencyCount tec;

-- 12. Emergency cost by payer category
SELECT 
    payerCategory, 
    SUM(TRY_CAST(baseEncounterCost AS FLOAT)) AS total_emergency_cost
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY payerCategory
ORDER BY total_emergency_cost DESC;

-- 13. Average duration of emergency encounters (in minutes) by type
SELECT 
    encounterType, 
    AVG(DATEDIFF(MINUTE, TRY_CAST(start AS DATETIME), TRY_CAST(stop AS DATETIME))) AS avg_duration_minutes
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY encounterType
ORDER BY avg_duration_minutes DESC;

-- 14. Emergency encounter frequency by hour of day
SELECT 
    DATEPART(HOUR, TRY_CAST(start AS DATETIME)) AS encounter_hour,
    COUNT(*) AS encounter_count
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY DATEPART(HOUR, TRY_CAST(start AS DATETIME))
ORDER BY encounter_hour;

-- 15. Seasonal trends in emergency admissions by month
SELECT 
    MONTH(TRY_CAST(start AS DATE)) AS month,
    COUNT(*) AS total_emergency_encounters
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY MONTH(TRY_CAST(start AS DATE))
ORDER BY month;

-- 16. Average time between consecutive emergency encounters per patient
WITH PatientEncounters AS (
    SELECT 
        patientID,
        encounterID,
        start,
        LEAD(start) OVER (PARTITION BY patientID ORDER BY start) AS next_encounter_start
    FROM Encounter
    WHERE encounterClass = 'Emergency'
)
SELECT 
    patientID,
    AVG(DATEDIFF(MINUTE, TRY_CAST(start AS DATETIME), TRY_CAST(next_encounter_start AS DATETIME))) AS avg_time_between_encounters
FROM PatientEncounters
WHERE next_encounter_start IS NOT NULL
GROUP BY patientID;

-- 17. Patient distribution by age group
WITH AgeGroups AS (
    SELECT 
        CASE 
            WHEN DATEDIFF(YEAR, TRY_CAST(birthday AS DATE), GETDATE()) < 18 THEN '0-17'
            WHEN DATEDIFF(YEAR, TRY_CAST(birthday AS DATE), GETDATE()) BETWEEN 18 AND 34 THEN '18-34'
            WHEN DATEDIFF(YEAR, TRY_CAST(birthday AS DATE), GETDATE()) BETWEEN 35 AND 64 THEN '35-64'
            ELSE '65+'
        END AS age_group
    FROM Patient
)
SELECT 
    age_group,
    COUNT(*) AS patient_count
FROM AgeGroups
GROUP BY age_group
ORDER BY patient_count DESC;

-- 18. Emergency encounter trends: Weekday vs Weekend
SELECT 
    CASE 
        WHEN DATEPART(WEEKDAY, TRY_CAST(start AS DATETIME)) IN (1, 7) THEN 'Weekend'
        ELSE 'Weekday'
    END AS day_type,
    COUNT(*) AS encounter_count
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY 
    CASE 
        WHEN DATEPART(WEEKDAY, TRY_CAST(start AS DATETIME)) IN (1, 7) THEN 'Weekend'
        ELSE 'Weekday'
    END
ORDER BY encounter_count DESC;

-- 19. Seasonal emergency encounter trends by month name
SELECT 
    DATENAME(MONTH, TRY_CAST(start AS DATETIME)) AS encounter_month,
    COUNT(*) AS emergency_encounters
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY DATENAME(MONTH, TRY_CAST(start AS DATETIME)), DATEPART(MONTH, TRY_CAST(start AS DATETIME))
ORDER BY DATEPART(MONTH, TRY_CAST(start AS DATETIME));

-- 20. Emergency admissions by age group
WITH AgeGroups AS (
    SELECT 
        e.encounterID,
        CASE 
            WHEN TRY_CAST(p.birthday AS DATE) IS NULL THEN 'Unknown'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) < 18 THEN '0-17'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) BETWEEN 18 AND 34 THEN '18-34'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) BETWEEN 35 AND 64 THEN '35-64'
            ELSE '65+'
        END AS age_group
    FROM Encounter e
    JOIN Patient p ON e.patientID = p.patientID
    WHERE e.encounterClass = 'Emergency'
)
SELECT 
    age_group,
    COUNT(*) AS encounter_count
FROM AgeGroups
GROUP BY age_group
ORDER BY encounter_count DESC;

-- 21. Emergency cost by age group
WITH PatientAgeGroups AS (
    SELECT 
        p.patientID,
        CASE 
            WHEN TRY_CAST(p.birthday AS DATE) IS NULL THEN 'Unknown'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) < 18 THEN '0-17'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) BETWEEN 18 AND 34 THEN '18-34'
            WHEN DATEDIFF(YEAR, TRY_CAST(p.birthday AS DATE), GETDATE()) BETWEEN 35 AND 64 THEN '35-64'
            ELSE '65+'
        END AS age_group
    FROM Patient p
)
SELECT 
    pag.age_group,
    AVG(TRY_CAST(e.baseEncounterCost AS FLOAT)) AS avg_emergency_cost
FROM Encounter e
JOIN PatientAgeGroups pag ON e.patientID = pag.patientID
WHERE 
    e.encounterClass = 'Emergency'
    AND ISNUMERIC(e.baseEncounterCost) = 1
GROUP BY pag.age_group
ORDER BY avg_emergency_cost DESC;

-- 22. Create temp table for emergency encounters
SELECT *
INTO #EmergencyEncounters
FROM Encounter
WHERE encounterClass = 'Emergency';

-- 23. Query temp table: Encounter counts by type
SELECT 
    encounterType, 
    COUNT(*) AS total_encounters
FROM #EmergencyEncounters
GROUP BY encounterType
ORDER BY total_encounters DESC;

-- 24. Create view for high-cost encounters
CREATE VIEW vw_HighCostEncounters AS
SELECT 
    encounterID,
    encounterClass,
    baseEncounterCost
FROM Encounter
WHERE TRY_CAST(baseEncounterCost AS FLOAT) > 1000;

-- 25. Emergency encounter count by age group (final query)
-- (reused query 20, no need to repeat here)

-- 26. Emergency cost by age group (final query)
-- (reused query 21, no need to repeat here)

-- 27. Create view for emergency encounter summary
CREATE VIEW vw_EmergencyEncounterSummary AS
SELECT 
    encounterClass,
    encounterType,
    COUNT(*) AS encounter_count,
    SUM(TRY_CAST(baseEncounterCost AS FLOAT)) AS total_emergency_cost
FROM Encounter
WHERE encounterClass = 'Emergency'
GROUP BY encounterClass, encounterType;

-- 28. Randomly assign existing patients to encounters
WITH RandomPatients AS (
    SELECT patientID, ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS rn
),
RandomEncounters AS (
    SELECT encounterID, ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS rn
)
UPDATE e
SET e.patientID = p.patientID
FROM Encounter e
JOIN RandomEncounters re ON e.encounterID = re.encounterID
JOIN RandomPatients p ON re.rn % (SELECT COUNT(*) FROM Patient) + 1 = p.rn;
