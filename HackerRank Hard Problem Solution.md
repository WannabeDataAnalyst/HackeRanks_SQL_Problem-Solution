-- Copy and paste into SQL in BigQuery. Excel files were created and uploaded to BigQuery to solve this problem. Those files are available under Excel Files. 

```
WITH sub_data AS 
(SELECT sub_date, hacker_id, SUM(Submissions) AS Total_Entries
FROM(
SELECT *,

COUNT(sub_id) OVER(
                 PARTITION BY hacker_id
                 ORDER BY sub_date
                 ROWS BETWEEN 0 PRECEDING AND CURRENT ROW 
) AS Submissions

FROM `joins-activity-331117.Hard_SQL_Q.Hard_SQL_Q_Data1` 
)
GROUP BY sub_date, hacker_id
ORDER BY sub_date, Total_Entries DESC, hacker_id
),
Unique_Entries AS
(
SELECT COUNT (Total_Entries) AS Entered,hacker_id, sub_date
FROM sub_data
GROUP BY sub_date, hacker_id
ORDER BY sub_date
),
Make_Case AS
(
SELECT  *, SUM(Entered) OVER (
                            PARTITION BY hacker_id
                            ORDER BY sub_date
                            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW 
) AS one_each_day

FROM Unique_Entries 
),
Count_Unique AS
(
SELECT sub_date, IF(one_each_day=1 AND sub_date="2016-03-01", "1", IF(one_each_day=2 AND sub_date="2016-03-02", "2", IF(one_each_day=3 AND sub_date="2016-03-03", "3", IF(one_each_day=4 AND sub_date="2016-03-04", "4", IF(one_each_day=5 AND sub_date="2016-03-05", "5", IF(one_each_day=6 AND sub_date="2016-03-06", "6", IF(one_each_day=7 AND sub_date="2016-03-07", "7", IF(one_each_day=8 AND sub_date="2016-03-08", "8", IF(one_each_day=9 AND sub_date="2016-03-09", "9", IF(one_each_day=10 AND sub_date="2016-03-10", "10", IF(one_each_day=11 AND sub_date="2016-03-11", "11", IF(one_each_day=12 AND sub_date="2016-03-12", "12", IF(one_each_day=13 AND sub_date="2016-03-13", "13", IF(one_each_day=14 AND sub_date="2016-03-14", "14", IF(one_each_day=15 AND sub_date="2016-03-15", "15", "" )))))))))))))))  AS unique_per_day
FROM Make_Case
),
Unique_Solution AS
(
SELECT COUNT(SAFE_CAST((unique_per_day) AS INT64)) AS unique_users, sub_date
FROM Count_Unique
GROUP BY sub_date
ORDER BY sub_date
),
max_subs AS
(SELECT sub_date, hacker_id, SUM(Submissions) AS Total_Entries

FROM(
SELECT *,

COUNT(sub_id) OVER(
                 PARTITION BY hacker_id
                 ORDER BY sub_date
                 ROWS BETWEEN 0 PRECEDING AND CURRENT ROW 
) AS Submissions

FROM `joins-activity-331117.Hard_SQL_Q.Hard_SQL_Q_Data1` 
)

GROUP BY sub_date, hacker_id
ORDER BY sub_date, Total_Entries DESC, hacker_id
),
AnotherT AS
(
SELECT sub_date, MIN(hacker_id) AS Min_Hack
FROM max_subs
WHERE  (Total_Entries, sub_date) IN 
(SELECT (MAX(Total_Entries), sub_date)
FROM max_subs
GROUP BY sub_date
)
GROUP BY sub_date
ORDER BY sub_date, Min_Hack
)
SELECT AnotherT.sub_date, unique_users, Min_Hack, name
FROM AnotherT 
JOIN `joins-activity-331117.Hard_SQL_Q.Hard_SQL_Q_Name_Id` AS HSN ON
    AnotherT.Min_Hack = HSN.hacker_id
JOIN Unique_Solution ON 
    AnotherT.sub_date = Unique_Solution.sub_date    

ORDER BY sub_date
```

