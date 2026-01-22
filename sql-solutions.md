**Q1. Level-7 Information School Subjects**  

**Problem**:  
Retrieve all Level-7 subjects offered by organisations classified as Schools whose name contains Information.  
**Approach**:
  * Filter subject codes matching level-7 pattern (XXXX7***)
  * Join subjects with organisational units and types
  * Ensure organisation type contains “School”
    
**Code**:      
```sql
CREATE OR REPLACE VIEW Q1(subject_code) AS
SELECT s.code
FROM subjects s
JOIN orgunits o ON s.offeredby = o.id
JOIN orgunit_types ot ON o.utype = ot.id
WHERE ot.name LIKE '%School%'
  AND o.longname LIKE '%Information%'
  AND s.code ~ '^[A-Z]{4}7[0-9]{3}$';
```

**Q2. COMP Courses Offering Only Lecture and Laboratory Classes**  

**Problem**:  
Find COMP courses that offer only Lecture and Laboratory class types.   
**Approach**:
  * Filter subjects with codes starting with COMP
  * Join courses, classes, and class types
  * Group by course and count distinct class types
  * Retain courses with exactly two class types: Lecture and Laboratory
       
**Code**:      
```sql
CREATE VIEW Q2(course_id)
AS
SELECT DISTINCT co.id 
FROM Courses co
JOIN Subjects sub ON co.subject = sub.id
JOIN Classes cl ON co.id = cl.course
JOIN Class_types clt ON cl.ctype = clt.id
LEFT JOIN 
   (
    SELECT DISTINCT co.id
    FROM Courses co 
    JOIN Classes cl ON co.id = cl.course 
    JOIN Class_types clt ON cl.ctype = clt.id 
    WHERE clt.name NOT IN ('Lecture', 'Laboratory')
   ) AS subs ON co.id = subs.id
WHERE sub.code LIKE 'COMP%'
AND subs.id IS NULL
GROUP BY co.id
HAVING COUNT(distinct clt.name)=2;
```

**Q3. High-Enrolment Students with Professor-Taught Courses**  

**Problem**:  
Retrieve UNSW IDs of students who:
  * Enrolled in at least five courses
  * Between 2008 and 2012
  * Courses had at least two professors
  * UNSW ID starts with 320
    
**Approach**:
  * Filter semesters by year range
  * Identify staff with title containing Prof
  * Ensure courses have two or more professors
  * Count qualifying enrolments per student
    
**Code**:      
```sql
CREATE VIEW Q3(unsw_id)
AS
SELECT p.unswid 
FROM people p
JOIN students std ON p.id = std.id
JOIN course_enrolments ce ON ce.student = std.id
JOIN courses co ON co.id = ce.course
JOIN semesters sem ON sem.id = co.semester
WHERE LEFT(CAST(p.unswid AS TEXT), 3) = '320'
AND sem.year BETWEEN 2008 AND 2012
AND co.id IN (
    SELECT co.id 
    FROM courses co 
    JOIN course_staff cs ON cs.course = co.id 
    JOIN staff sf ON cs.staff = sf.id
    JOIN people p ON p.id = sf.id
    WHERE p.title = 'Prof'
    GROUP BY co.id
    HAVING COUNT(p.id) >= 2
)
GROUP BY p.unswid
HAVING COUNT(ce.course) >= 5;
```



