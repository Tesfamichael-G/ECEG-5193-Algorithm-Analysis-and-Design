# Chapter 6 — Structured Query Language (SQL)

!!! abstract "Chapter at a glance"
    **Coverage:** objectives of SQL · DDL (CREATE, ALTER, DROP) · DML (SELECT, joins, aggregates, subqueries, set operations, EXISTS, NULLs) · INSERT, UPDATE, DELETE.

---

## Learning outcomes

1. Create, alter, and drop databases and tables with constraints.
2. Write SELECT queries with WHERE, ORDER BY, DISTINCT, aliases.
3. Use joins, aggregates, GROUP BY, HAVING, and subqueries correctly.
4. Modify data with INSERT, UPDATE, DELETE while respecting constraints.

---

## 6.1 Objectives of SQL

SQL was designed to be:

- **Declarative** — specify *what* you want, not *how* to get it.
- **Universal** — the same syntax works on every major DBMS.
- **Standardised** — ANSI/ISO standard since 1986 (SQL-86, SQL-92, SQL:1999, SQL:2003, SQL:2008, SQL:2011, …).
- **Non-procedural** — queries express sets of rows, not loops.

Two sublanguages:

- **DDL** (Data Definition Language) — define schema.
- **DML** (Data Manipulation Language) — query and modify data.

---

## 6.2 DDL — CREATE, ALTER, DROP

```sql
CREATE DATABASE COMPANY;

CREATE TABLE DEPARTMENT (
    dnumber INT PRIMARY KEY,
    dname   VARCHAR(50) NOT NULL UNIQUE,
    mgr_ssn CHAR(9)
);

CREATE TABLE EMPLOYEE (
    ssn       CHAR(9) PRIMARY KEY,
    name      VARCHAR(50) NOT NULL,
    salary    DECIMAL(10, 2) CHECK (salary >= 0),
    dno       INT,
    FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber)
);

ALTER TABLE DEPARTMENT
    ADD FOREIGN KEY (mgr_ssn) REFERENCES EMPLOYEE(ssn)
    ON DELETE SET NULL ON UPDATE CASCADE;

DROP TABLE EMPLOYEE;
```

**Constraints summary:**

| Constraint | Meaning |
|---|---|
| `PRIMARY KEY` | Uniquely identifies each tuple; implies NOT NULL |
| `FOREIGN KEY` | Referential integrity |
| `UNIQUE` | All values distinct (NULL allowed) |
| `NOT NULL` | No NULL allowed |
| `CHECK` | Boolean condition on each tuple |
| `DEFAULT` | Default value |

**Referential actions:**

- `ON DELETE RESTRICT` — reject the delete (default in many DBMS).
- `ON DELETE CASCADE` — delete dependent tuples.
- `ON DELETE SET NULL` — set foreign key to NULL.
- `ON DELETE SET DEFAULT` — set to default.

---

## 6.3 DML — SELECT

### 6.3.1 Basic form

```sql
SELECT   DISTINCT dno
FROM     EMPLOYEE
WHERE    salary > 40000
ORDER BY name ASC;
```

**Logical evaluation order:**

1. `FROM`
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `DISTINCT`
7. `ORDER BY`

(This is *why* an aggregate cannot appear in `WHERE` — aggregation has not happened yet.)

### 6.3.2 Joins

**Explicit inner join:**

```sql
SELECT e.name, d.dname
FROM   EMPLOYEE e INNER JOIN DEPARTMENT d ON e.dno = d.dnumber;
```

**Left outer join** (keep employees without a department):

```sql
SELECT e.name, d.dname
FROM   EMPLOYEE e LEFT JOIN DEPARTMENT d ON e.dno = d.dnumber;
```

### 6.3.3 Aggregates and grouping

```sql
SELECT   dno, AVG(salary) AS avg_salary, COUNT(*) AS n_employees
FROM     EMPLOYEE
GROUP BY dno
HAVING   COUNT(*) > 5;
```

**Rules.**

- Aggregate functions: `COUNT, SUM, AVG, MIN, MAX`.
- Every non-aggregated column in `SELECT` must appear in `GROUP BY`.
- `WHERE` filters rows **before** grouping; `HAVING` filters groups **after** grouping.

### 6.3.4 Subqueries

**Scalar subquery (one value):**

```sql
SELECT name
FROM   EMPLOYEE
WHERE  salary > (SELECT AVG(salary) FROM EMPLOYEE);
```

**Correlated subquery:**

```sql
SELECT name
FROM   EMPLOYEE e
WHERE  salary > (SELECT AVG(salary) FROM EMPLOYEE WHERE dno = e.dno);
```

**EXISTS:**

```sql
SELECT d.dname
FROM   DEPARTMENT d
WHERE  EXISTS (SELECT 1 FROM EMPLOYEE e WHERE e.dno = d.dnumber);
```

**IN vs. EXISTS.** `IN` compares a value to a subquery result; `EXISTS` tests whether the subquery produces any rows. Performance depends on the DBMS and the indexes.

### 6.3.5 Set operations

```sql
SELECT ssn FROM EMPLOYEE
UNION
SELECT ssn FROM MANAGER;

SELECT ssn FROM EMPLOYEE
INTERSECT
SELECT ssn FROM MANAGER;

SELECT ssn FROM EMPLOYEE
EXCEPT              -- MINUS in Oracle
SELECT ssn FROM MANAGER;
```

**`UNION`** removes duplicates; **`UNION ALL`** keeps them (faster).

### 6.3.6 NULL handling

- `NULL` is **unknown**, not empty and not zero.
- `NULL = NULL` evaluates to `UNKNOWN`, not `TRUE`.
- Use `IS NULL` / `IS NOT NULL` for comparison.
- Arithmetic with `NULL` yields `NULL`.
- `COUNT(column)` ignores NULLs; `COUNT(*)` counts rows.

### 6.3.7 String operations

```sql
SELECT name
FROM   EMPLOYEE
WHERE  name LIKE 'Sm%'             -- starts with Sm
   OR  name LIKE '_mith'           -- _ matches one character
   OR  SUBSTRING(name, 1, 3) = 'Ali';
```

---

## Worked example 6.1 — six SQL queries

**Schema.** `EMPLOYEE(ssn, name, salary, dno)`, `DEPARTMENT(dnumber, dname, mgr_ssn)`, `PROJECT(pnumber, pname, dnum)`, `WORKS_ON(essn, pno, hours)`.

**Q1.** Names of employees earning more than the average.

```sql
SELECT name FROM EMPLOYEE
WHERE  salary > (SELECT AVG(salary) FROM EMPLOYEE);
```

**Q2.** Departments with more than 5 employees and an average salary of at least 40,000.

```sql
SELECT   d.dname
FROM     DEPARTMENT d JOIN EMPLOYEE e ON e.dno = d.dnumber
GROUP BY d.dnumber, d.dname
HAVING   COUNT(*) > 5 AND AVG(e.salary) >= 40000;
```

**Q3.** Names of employees who do not work on any project.

```sql
SELECT name FROM EMPLOYEE e
WHERE  NOT EXISTS (SELECT 1 FROM WORKS_ON w WHERE w.essn = e.ssn);
```

**Q4.** Names of employees who work on all projects.

```sql
SELECT name FROM EMPLOYEE e
WHERE  NOT EXISTS (
    SELECT pnumber FROM PROJECT
    EXCEPT
    SELECT pno      FROM WORKS_ON WHERE essn = e.ssn
);
```

**Q5.** For each department, the name of its highest-paid employee.

```sql
SELECT d.dname, e.name, e.salary
FROM   EMPLOYEE e
       JOIN DEPARTMENT d ON e.dno = d.dnumber
WHERE  e.salary = (SELECT MAX(salary) FROM EMPLOYEE WHERE dno = d.dnumber);
```

**Q6.** Update salaries — give every employee in department 5 a 10% raise.

```sql
UPDATE EMPLOYEE
SET    salary = salary * 1.10
WHERE  dno = 5;
```

---

## 6.4 INSERT, UPDATE, DELETE

```sql
INSERT INTO EMPLOYEE (ssn, name, salary, dno)
VALUES ('999999999', 'Alice', 45000, 5);

UPDATE EMPLOYEE SET salary = salary * 1.05 WHERE dno = 5;

DELETE FROM EMPLOYEE WHERE ssn = '999999999';
```

**Constraint violation examples:**

- Insert with duplicate primary key → `UNIQUE` violation.
- Insert with non-existent `dno` → `FOREIGN KEY` violation.
- Delete a department that still has employees → `FOREIGN KEY` violation (unless `ON DELETE CASCADE`).
- Update a primary key that other rows reference → `FOREIGN KEY` violation (unless `ON UPDATE CASCADE`).

---

## 6.5 Quiz

1. Write the DDL for `EMPLOYEE` and `DEPARTMENT` with all necessary constraints.
2. What is the logical evaluation order of a SELECT query?
3. Explain the difference between `WHERE` and `HAVING`.
4. Write a query that finds employees who earn more than their department average.
5. What is the difference between `IN` and `EXISTS`? Give an example where each is preferred.

---

## Answer key

??? success "Click to reveal"

    1. (See §6.2.)
    2. FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY.
    3. WHERE filters rows before grouping; HAVING filters groups after grouping.
    4. (Q6.1 above, using correlated subquery.)
    5. IN checks against a set of values; EXISTS tests for existence of matching rows. EXISTS is often faster with correlated subqueries and can short-circuit; IN is simpler when the subquery returns a static list.

---

## Further reading

- Elmasri & Navathe, §4.1–§4.5, §5.1–§5.4.
- Silberschatz et al., §3.1–§3.10, §4.1–§4.2.
- SQLBolt (sqlbolt.com) for hands-on practice.