SQL 
SIMPLEST SQL TEMPLATE
```
SELECT * FROM CITY
WHERE COUNTRYCODE = 'USA' AND POPULATION>100000 OR NOT LIKE '%[a, e, i, o, u]'
ORDER BY EMPLOYEE_ID ASC
```

SIMPLE TEMPLATE
```
SELECT DISTINCT NAME 
FROM CITY c INNER JOIN State s
ON c.country = s.country
WHERE COUNTRYCODE = 'USA' AND POPULATION>100000 OR NOT LIKE '%[a, e, i, o, u]'
GROUP BY c.country
ORDER BY EMPLOYEE_ID ASC
```

ADVANCE SQL TEMPLATE (SUBQUERY)
```
SELECT c.hacker_id as id, h.name as name, COUNT(c.hacker_id) as challenges_created
FROM Challenges c INNER JOIN Hackers h
ON c.hacker_id = h.hacker_id
GROUP BY id, name
HAVING challenges_created = (
    SELECT MAX(counter)
    FROM (
        SELECT c.hacker_id as id, COUNT(c.hacker_id) as counter
        FROM Challenges c
        GROUP BY id
    ) as _
) or challenges_created in (
    SELECT counter
    FROM (
        SELECT c.hacker_id as id, COUNT(c.hacker_id) as counter
        FROM Challenges c
        GROUP BY id
    ) as _
    GROUP BY counter
    HAVING COUNT(counter) = 1
)
ORDER BY challenges_created DESC, id
```
```
SELECT H.HACKER_ID, H.NAME
	FROM SUBMISSIONS S
	JOIN HACKERS H 
		ON H.HACKER_ID = S.HACKER_ID
	JOIN CHALLENGES C 
		ON C.CHALLENGE_ID = S.CHALLENGE_ID
	JOIN DIFFICULTY D 
		ON D.DIFFICULTY_LEVEL = C.DIFFICULTY_LEVEL
	WHERE S.SCORE = D.SCORE
		AND C.DIFFICULTY_LEVEL = D.DIFFICULTY_LEVEL
	GROUP BY H.HACKER_ID, H.NAME
		HAVING COUNT(s.hacker_id) > 1
	ORDER BY COUNT(S.HACKER_ID) DESC, S.HACKER_ID ASC
```
```
SELECT W.ID, P.AGE, W.COINS_NEEDED, W.POWER 
FROM WANDS W, WANDS_PROPERTY P 
WHERE W.CODE = P.CODE
    AND P.IS_EVIL = 0
    AND W.COINS_NEEDED = (SELECT MIN(COINS_NEEDED) 
                          FROM WANDS W1, WANDS_PROPERTY P1 
                          WHERE W1.CODE = P1.CODE
                              AND W1.POWER = W.POWER
                              AND P1.AGE = P.AGE) 
ORDER BY W.POWER DESC, P.AGE DESC
```
```
SELECT H.HACKER_ID, NAME, SUM(SCORE) AS TOTAL_SCORE
FROM HACKERS H 
JOIN
/* FIND MAX_SCORE*/
(SELECT HACKER_ID, CHALLENGE_ID, MAX(SCORE) AS SCORE FROM SUBMISSIONS GROUP BY CHALLENGE_ID, HACKER_ID) MAX_SCORE

ON H.HACKER_ID=MAX_SCORE.HACKER_ID
GROUP BY H.HACKER_ID, NAME

/* DON'T ACCEPT HACKERS WITH TOTAL_SCORE=0 */
HAVING TOTAL_SCORE > 0

/* FINALLY ORDER AS REQUIRED */
ORDER BY TOTAL_SCORE DESC, H.HACKER_ID
```
```
SELECT Start_Date, MIN(End_Date)
FROM 
/* Choose start dates that are not end dates of other projects (if a start date is an end date, it is part of the samee project) */
    (SELECT Start_Date FROM Projects WHERE Start_Date NOT IN (SELECT End_Date FROM Projects)) a,
/* Choose end dates that are not end dates of other projects */
    (SELECT end_date FROM PROJECTS WHERE end_date NOT IN (SELECT start_date FROM PROJECTS)) b
/* At this point, we should have a list of start dates and end dates that don't necessarily correspond with each other */
/* This makes sure we only choose end dates that fall after the start date, and choosing the MIN means for the particular start_date, we get the closest end date that does not coincide with the start of another task */
where start_date < end_date
GROUP BY start_date
ORDER BY datediff(start_date, MIN(end_date)) DESC, start_date
```

ADVANCE FUNCTION WITH VARIABLE (calculating median)
```
SET @N := 0;
SELECT COUNT(*) FROM STATION INTO @TOTAL;
SELECT ROUND(AVG(A.LAT_N), 4)
FROM (SELECT @N := @N +1 AS ROW_ID, LAT_N 
      FROM STATION 
      ORDER BY LAT_N) A
WHERE
    CASE WHEN MOD(@TOTAL, 2) = 0 
            THEN A.ROW_ID IN (@TOTAL/2, (@TOTAL/2+1))
            ELSE A.ROW_ID = (@TOTAL+1)/2
    END
;

```

UNIQUE/DISTINCT
```
SELECT DISTINCT CITY FROM STATION
WHERE ID % 2 = 0
```

# AGGREGATION
## DIFFERENCE IN COUNT (ONLY ONE SELECT)
```
SELECT COUNT(CITY) - COUNT(DISTINCT CITY) FROM STATION;
```

## SUM/AVG (Sum/Average)
```
SELECT SUM(POPULATION)
FROM CITY
WHERE DISTRICT='CALIFORNIA'
GROUP BY DISTRICT
```

## FLOOR/ROUND/CEIL (round down, round up, floor, ceiling, rounding)
```
SELECT FLOOR(AVG(POPULATION))
FROM CITY
GROUP BY DISTRICT
```

Rount to decimals
```
SELECT ROUND(SUM(LAT_N),2),ROUND(SUM(LONG_W),2) 
FROM STATION
```

## LIMIT
```
SELECT MONTHS*SALARY AS EARNINGS, COUNT(*) 
FROM EMPLOYEE 
GROUP BY 1 
ORDER BY EARNINGS DESC LIMIT 1;
```

# JOIN
## EQUI-JOIN
```
SELECT SUM(CITY.POPULATION) 
FROM CITY, COUNTRY
WHERE CITY.COUNTRYCODE = COUNTRY.CODE AND COUNTRY.CONTINENT = 'Asia';
```

## INNER JOIN (USING)
```
-- Select S.Name
-- From ( Students S join Friends F Using(ID)
--        join Packages P1 on S.ID=P1.ID
--        join Packages P2 on F.Friend_ID=P2.ID)
-- Where P2.Salary > P1.Salary
-- Order By P2.Salary;
Select S.Name
From Students s 
JOIN Friends f on s.id = f.id
JOIN Packages p1 on p1.id = f.id
JOIN Packages p2 on p2.id = f.Friend_ID
Where P2.Salary > P1.Salary
Order By P2.Salary;
```

SUBQUERY SELECT
```
SELECT ROUND(LONG_W,4) 
FROM STATION 
WHERE LAT_N = (
    SELECT MAX(LAT_N) 
	FROM STATION 
	WHERE LAT_N<137.2345);
```

ABS (Absolute)
```
SELECT ROUND( ABS(MIN(LAT_N)-MAX(LAT_N)) + ABS(MIN(LONG_W)-MAX(LONG_W) ),4) 
FROM STATION;
```

MEDIAN
```
SELECT ROUND(S.LAT_N,4)
FROM STATION S 
WHERE (
    SELECT COUNT(LAT_N) 
    FROM STATION 
    WHERE LAT_N < S.LAT_N ) = (
        SELECT COUNT(LAT_N) 
        FROM STATION 
        WHERE LAT_N > S.LAT_N)
```

OFFSET
```
SELECT ROUND(LAT_N,4)
FROM STATION 
ORDER BY LAT_N
LIMIT 1 OFFSET 249
```

POWER
```
SELECT
    ROUND(SQRT(
        POWER(MAX(LAT_N)  - MIN(LAT_N),  2)
      + POWER(MAX(LONG_W) - MIN(LONG_W), 2)
    ), 4)
FROM 
    STATION;
```

ORDER BY/LENGTH
```
SELECT CITY, LENGTH(CITY) FROM STATION
ORDER BY LENGTH(CITY),CITY ASC
LIMIT 1;

SELECT CITY, LENGTH(CITY) FROM STATION
ORDER BY LENGTH(CITY) DESC
LIMIT 1;
```

REPLACE
```
SELECT CEIL(AVG(Salary)-AVG(REPLACE(Salary,'0',''))) FROM EMPLOYEES;
```

LIKE
1. Starts with 'a%'
2. Ends with '%a'
3. any position '%or%'
4. second position '_r%'
5. like this supports index '[hH][oO][sS][pP][iI][tT][aA][lL][iI][sS][tT]%'
6. WHERE LOWER(Name) LIKE '%hospitalist%' 

```
SELECT DISTINCT CITY FROM STATION 
WHERE CITY LIKE '[a, e, i, o, u]%' 

DOES NOT BEGIN AND DOES NOT END WITH VOWELS
SELECT DISTINCT CITY FROM STATION
WHERE CITY LIKE '[^aeiou]%[^aeiou]';
```

REGEXP(MYSQL) MSSQL EXTENDS LIKE
1. ^ beginning
2. [^]negated set
3. $ end of line
4. | or
5. . any character except line breaks
6. * match 0 or more quantifier followed after #5

```
BEGIN WITH VOWELS
SELECT DISTINCT CITY FROM STATION 
WHERE CITY REGEXP "^[aeiou].*";

DOES NOT BEGIN WITH VOWELS OR DOES NOT END WITH VOWELS
SELECT DISTINCT CITY FROM STATION
WHERE CITY REGEXP '^[^aeiou]|[^aeiou]$';

DOES NOT BEGIN AND DOES NOT END WITH VOWELS
SELECT DISTINCT CITY FROM STATION WHERE CITY REGEXP '^[^aeiou].*[^aeiou]$';
```

LEFT/RIGHT IN LIST (SUBSTRING) (PART OF WORD)
```
SELECT DISTINCT CITY FROM STATION WHERE RIGHT(CITY,1) IN ('a','i','e','o','u');
```

NEGATE/NOT
```
SELECT DISTINCT CITY FROM STATION 
WHERE CITY NOT LIKE '[a, e, i, o, u]%' 
```

OR
```
SELECT DISTINCT CITY FROM STATION 
WHERE CITY NOT LIKE '%[a, e, i, o, u]' OR NOT LIKE '[a, e, i, o, u]%'
```

SORT/ORDER BY
```
SELECT NAME FROM STUDENTS
WHERE MARKS > 75
ORDER BY RIGHT(NAME, 3), ID ASC
```

CONCAT/SUBSTRING
```
SELECT CONCAT(NAME, '(', SUBSTR(OCCUPATION,1,1), ')')
FROM OCCUPATIONS
ORDER BY NAME ASC;

SELECT CONCAT('THERE ARE A TOTAL OF ', COUNT(1), ' ' , LOWER(OCCUPATION), 'S.')
FROM OCCUPATIONS
GROUP BY OCCUPATION
ORDER BY COUNT(1) ASC, OCCUPATION ASC;
```

CASE (IF CONDITION)
```
SELECT CASE
    WHEN P IS NULL THEN CONCAT(N, ' Root')
    WHEN N IN (SELECT DISTINCT P FROM BST) THEN CONCAT(N, ' Inner')
    ELSE CONCAT(N, ' Leaf')
    END
FROM BST
ORDER BY N ASC
```
```
SELECT CASE
    WHEN A + B <= C OR A + C <= B AND B + C <= A THEN 'Not A Triangle'
    WHEN A = B AND B = C THEN 'Equilateral'
    WHEN A = B OR B = C OR A = C THEN 'Isosceles'
    ELSE 'Scalene'
    END
FROM TRIANGLES
```

IF/BETWEEN (STRICT IF)
```
SELECT IF(GRADE<8, NULL, NAME), GRADE, MARKS
FROM STUDENTS, GRADES
WHERE MARKS BETWEEN MIN_MARK AND MAX_MARK
ORDER BY GRADE DESC, NAME ASC
```

RANK/PARTITION
```
SELECT 
MIN(CASE WHEN A.OCCUPATION = 'DOCTOR' THEN A.NAME ELSE NULL END) AS DOCTOR, 
MIN(CASE WHEN A.OCCUPATION = 'PROFESSOR' THEN A.NAME ELSE NULL END) AS PROFESSOR, 
MIN(CASE WHEN A.OCCUPATION = 'SINGER' THEN A.NAME ELSE NULL END) AS SINGER, 
MIN(CASE WHEN A.OCCUPATION = 'ACTOR' THEN A.NAME ELSE NULL END) AS ACTOR 
FROM (
    SELECT OCCUPATION, NAME , RANK() OVER (
        PARTITION BY OCCUPATION 
        ORDER BY NAME ASC
    ) ROWNUMBER
    FROM OCCUPATIONS) AS A 
GROUP BY ROWNUMBER
```

ADVANCE SELECT
```
select C.company_code, 
       C.founder,
       count(distinct L.lead_manager_code),
       count(distinct S.senior_manager_code),
       count(distinct M.manager_code),
       count(distinct E.employee_code)
                            from Company as C,
                                 Lead_Manager as L,
                                 Senior_Manager as S,
                                 Manager as M,
                                 Employee as E 
                            WHERE E.manager_code = M.manager_code 
                                  AND M.senior_manager_code = S.senior_manager_code
                                  AND L.lead_manager_code = S.lead_manager_code
                                  AND C.company_code = L.company_code
                            group by C.company_code, C.founder
                            order by C.company_code
```