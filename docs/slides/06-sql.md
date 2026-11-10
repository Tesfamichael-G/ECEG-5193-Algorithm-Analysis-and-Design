---
revealjs:
  mode: diapo
  highlight:
    light: vs
    dark:  vs2015
  autoAnimate:  true
  autoAnimateDuration: 1.0
  transition: slide
  transitionSpeed: default
  js: assets/lecture-06.js

---

# ECEG-4191

## Chapter 6 — Structured Query Language (SQL)

---

## Learning outcomes

<div class="fragment">

1. Create schemas with **DDL**: `CREATE`, `ALTER`, `DROP`, all constraint types.

</div>

<div class="fragment">

2. Write **SELECT** queries with filtering, projection, joins, aggregates, subqueries.

</div>

<div class="fragment">

3. Use **GROUP BY** and **HAVING** correctly.

</div>

<div class="fragment">

4. Handle **NULL** semantics, set operations, and `EXISTS`.

</div>

<div class="fragment">

5. Modify data with **INSERT**, **UPDATE**, **DELETE** while respecting constraints.

</div>

---

## 6.1 — Objectives of SQL

???+ note "Design goals"
    - **Declarative** — specify *what*, not *how*.
    - **Universal** — one syntax, every major DBMS.
    - **Standardised** — ANSI/ISO since 1986 (SQL-86, 92, 99, 2003, 2008, 2011, 2016, 2023).
    - **Non-procedural** — queries return sets, not loops.

<div class="fragment">

???+ info "Two sublanguages"
    | Sublanguage | Purpose | Statements |
    |---|---|---|
    | **DDL** | Define / modify schema | `CREATE`, `ALTER`, `DROP` |
    | **DML** | Query / modify data | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |

</div>

<div class="fragment">

!!! tip "SQL has three modes of execution"
    1. **Interactive** — typed at a terminal.
    2. **Embedded** — called from a host language (Java, Python).
    3. **Dynamic** — constructed at runtime (ORM frameworks).

</div>

---

## 6.2 — DDL: Creating a Table

```sql
CREATE TABLE DEPARTMENT (
    dnumber INT PRIMARY KEY,
    dname   VARCHAR(50) NOT NULL UNIQUE,
    mgr_ssn CHAR(9)
);
```

<div class="fragment">

???+ note "The constraint vocabulary"
    | Constraint | Meaning |
    |---|---|
    | `PRIMARY KEY` | Uniquely identifies each tuple; **implies NOT NULL** |
    | `FOREIGN KEY` | Referential integrity |
    | `UNIQUE` | All values distinct (NULL allowed) |
    | `NOT NULL` | No NULL values |
    | `CHECK` | Boolean condition on each tuple |
    | `DEFAULT` | Default value if none provided |

</div>

<div class="fragment">

!!! warning "PRIMARY KEY implies UNIQUE + NOT NULL"
    You do not need to write both. `PRIMARY KEY` already enforces both.

</div>

---

## 6.3 — DDL: Foreign Keys

```sql
CREATE TABLE EMPLOYEE (
    ssn     CHAR(9) PRIMARY KEY,
    name    VARCHAR(50) NOT NULL,
    salary  DECIMAL(10,2) CHECK (salary >= 0),
    dno     INT,
    FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

<div class="fragment">

???+ note "Referential actions"
    | Action | On DELETE / UPDATE |
    |---|---|
    | `RESTRICT` / `NO ACTION` | Reject the operation (default) |
    | `CASCADE` | Propagate the change |
    | `SET NULL` | Set the foreign key to NULL |
    | `SET DEFAULT` | Set to default |

</div>

<div class="fragment">

???+ info "Combining actions"
    You can specify different actions for delete and update:
    ```sql
    FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber)
        ON DELETE SET NULL
        ON UPDATE CASCADE
    ```

</div>

---

## 6.4 — DDL: Circular References

```sql
CREATE TABLE DEPARTMENT (
    dnumber INT PRIMARY KEY,
    dname   VARCHAR(50),
    mgr_ssn CHAR(9)               -- no FK here yet
);

CREATE TABLE EMPLOYEE (
    ssn  CHAR(9) PRIMARY KEY,
    name VARCHAR(50),
    dno  INT,
    FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber)
);

ALTER TABLE DEPARTMENT
    ADD FOREIGN KEY (mgr_ssn) REFERENCES EMPLOYEE(ssn);
```

<div class="fragment">

!!! tip "The circular-reference pattern"
    - Create the first table **without** the foreign key.
    - Create the second table with its foreign key.
    - Add the first table's foreign key with `ALTER TABLE`.

</div>

<div class="fragment">

???+ warning "The alternative — deferred constraints"
    Some DBMSes support `DEFERRABLE INITIALLY DEFERRED` — the constraint is checked at commit time, not statement time. This allows circular references without ALTER.

</div>

---

## 6.5 — ALTER and DROP

```sql
ALTER TABLE EMPLOYEE ADD COLUMN email VARCHAR(100);
ALTER TABLE EMPLOYEE DROP COLUMN email;
ALTER TABLE EMPLOYEE ADD CONSTRAINT chk_salary CHECK (salary >= 1000);
ALTER TABLE EMPLOYEE DROP CONSTRAINT chk_salary;

DROP TABLE EMPLOYEE;                    -- fails if other tables reference it
DROP TABLE EMPLOYEE CASCADE;            -- drops dependent constraints too
DROP TABLE IF EXISTS EMPLOYEE;          -- no error if missing
```

<div class="fragment">

!!! warning "ALTER with data present"
    Adding a `NOT NULL` column to a table with rows requires a **default value**:
    ```sql
    ALTER TABLE EMPLOYEE ADD COLUMN status VARCHAR(10) NOT NULL DEFAULT 'active';
    ```
    Without a default, the statement fails.

</div>

<div class="fragment">

???+ info "DROP vs TRUNCATE vs DELETE"
    | Command | Removes |
    |---|---|
    | `DROP TABLE` | The table and all its data |
    | `TRUNCATE TABLE` | All rows, keeps the schema |
    | `DELETE FROM` | Selected rows |

</div>

---

## 6.6 — SELECT: The Basic Form

```sql
SELECT   column1, column2, ...
FROM     table1, table2, ...
WHERE    condition
GROUP BY grouping_columns
HAVING   group_condition
ORDER BY sort_columns;
```

<div class="fragment">

???+ note "Logical evaluation order"
    1. `FROM` — build the Cartesian product of the tables.
    2. `WHERE` — filter tuples.
    3. `GROUP BY` — group remaining tuples.
    4. `HAVING` — filter groups.
    5. `SELECT` — evaluate the projected columns.
    6. `DISTINCT` — remove duplicate rows.
    7. `ORDER BY` — sort the result.

</div>

<div class="fragment">

!!! warning "The evaluation order has consequences"
    - Aggregates cannot appear in `WHERE` (aggregation hasn't happened yet).
    - Column aliases defined in `SELECT` cannot be used in `WHERE`.
    - `ORDER BY` can use `SELECT` aliases.

</div>

---

## 6.7 — Basic SELECT Examples

```sql
-- All employees
SELECT * FROM EMPLOYEE;

-- Filter
SELECT ssn, name FROM EMPLOYEE WHERE salary > 40000;

-- Distinct departments
SELECT DISTINCT dno FROM EMPLOYEE;

-- Sort
SELECT name, salary FROM EMPLOYEE ORDER BY salary DESC, name ASC;

-- Alias
SELECT name AS employee_name FROM EMPLOYEE;
```

<div class="fragment">

???+ info "`DISTINCT` vs no `DISTINCT`"
    - `SELECT dno FROM EMPLOYEE` — one row per employee (duplicates).
    - `SELECT DISTINCT dno FROM EMPLOYEE` — one row per department.

</div>

<div class="fragment">

!!! tip "SQL is bag semantics by default"
    Unlike relational algebra (set), SQL returns **bags** — duplicates preserved unless `DISTINCT` is specified. This is a frequent source of confusion.

</div>

---

## 6.8 — Joins in SQL

???+ note "Four join syntaxes"
    ```sql
    -- Implicit (old style)
    SELECT e.name, d.dname
    FROM   EMPLOYEE e, DEPARTMENT d
    WHERE  e.dno = d.dnumber;

    -- Explicit INNER JOIN
    SELECT e.name, d.dname
    FROM   EMPLOYEE e INNER JOIN DEPARTMENT d ON e.dno = d.dnumber;

    -- LEFT JOIN (preserves employees with no dept)
    SELECT e.name, d.dname
    FROM   EMPLOYEE e LEFT JOIN DEPARTMENT d ON e.dno = d.dnumber;

    -- NATURAL JOIN (equal common columns)
    SELECT e.name, d.dname
    FROM   EMPLOYEE e NATURAL JOIN DEPARTMENT d;
    ```

<div class="fragment">

???+ info "Join types"
    | SQL | Equivalent to |
    |---|---|
    | `INNER JOIN` | Natural join / equijoin |
    | `LEFT JOIN` | Left outer join |
    | `RIGHT JOIN` | Right outer join |
    | `FULL OUTER JOIN` | Full outer join |

</div>

<div class="fragment">

!!! warning "Implicit join pitfall"
    Forgetting the `WHERE` clause in the old-style syntax produces a **Cartesian product** — for 1M employees × 1k departments, that's 10⁹ rows.

</div>

---

## 6.9 — Aggregates and GROUP BY

```sql
SELECT   dno, COUNT(*) AS n, AVG(salary) AS avg_sal
FROM     EMPLOYEE
GROUP BY dno
HAVING   COUNT(*) > 5;
```

<div class="fragment">

???+ note "Aggregate functions"
    | Function | Returns |
    |---|---|
    | `COUNT(*)` | Number of rows |
    | `COUNT(col)` | Number of non-NULL values in `col` |
    | `SUM(col)` | Sum of non-NULL values |
    | `AVG(col)` | Average of non-NULL values |
    | `MIN(col)`, `MAX(col)` | Minimum, maximum |

</div>

<div class="fragment">

!!! warning "The GROUP BY rule"
    Every column in `SELECT` that is **not** an aggregate must appear in `GROUP BY`. Violating this is a syntax error in strict SQL mode and undefined behavior otherwise.

</div>

<div class="fragment">

???+ info "WHERE vs HAVING"
    - `WHERE` filters **rows** before grouping.
    - `HAVING` filters **groups** after grouping.
    - `HAVING` can use aggregate functions; `WHERE` cannot.

</div>

---

## Worked example 6.1 — Aggregates

!!! tip "Problem:"
    For each department, list `dname`, number of employees, and average salary, but only for departments with more than 5 employees and average salary ≥ 40,000.

<div class="fragment">

???+ note "**Step 1 — the query.**"
    ```sql
    SELECT   d.dname, COUNT(*) AS n, AVG(e.salary) AS avg_sal
    FROM     DEPARTMENT d JOIN EMPLOYEE e ON e.dno = d.dnumber
    GROUP BY d.dnumber, d.dname
    HAVING   COUNT(*) > 5 AND AVG(e.salary) >= 40000;
    ```

</div>

<div class="fragment">

??? info "**Step 2 — why `d.dnumber` is in `GROUP BY`.**"
    `dname` is functionally determined by `dnumber`, so grouping by both is valid. Strictly, grouping by `dnumber` alone is enough (in PostgreSQL, you can omit `dname`), but grouping by both is portable.

</div>

<div class="fragment">

??? success "**Step 3 — the semantics.**"
    - `JOIN` pairs employees with their departments.
    - `GROUP BY` groups employees per department.
    - `HAVING` discards departments that fail either condition.

</div>

---

## 6.10 — Subqueries

???+ note "Three kinds of subquery"
    | Kind | Returns | Used in |
    |---|---|---|
    | **Scalar** | One value | Comparison (`=`, `<`, `>`) |
    | **Row** | One row | Row comparison |
    | **Table** | Many rows | `IN`, `EXISTS`, `ANY`, `ALL` |

<div class="fragment">

???+ info "Scalar subquery example"
    ```sql
    SELECT name FROM EMPLOYEE
    WHERE salary > (SELECT AVG(salary) FROM EMPLOYEE);
    ```
    
    The subquery returns one number (the company average); the outer query compares against it.

</div>

<div class="fragment">

???+ info "Correlated subquery"
    ```sql
    SELECT name FROM EMPLOYEE e
    WHERE salary > (SELECT AVG(salary) FROM EMPLOYEE WHERE dno = e.dno);
    ```
    
    The subquery references `e.dno` from the outer query — it is **re-executed per outer row**.

</div>

<div class="fragment">

!!! warning "Correlated subqueries are expensive"
    Naïve execution is $O(n \times m)$ — one subquery per outer row. Query optimizers often rewrite them as joins.

</div>

---

## Worked example 6.2 — Subquery problems

!!! tip "Problems:"

<div class="fragment">

???+ note "**Q1 — employees earning above the company average**"
    ```sql
    SELECT name FROM EMPLOYEE
    WHERE salary > (SELECT AVG(salary) FROM EMPLOYEE);
    ```

</div>

<div class="fragment">

??? info "**Q2 — employees earning above their department average**"
    ```sql
    SELECT name FROM EMPLOYEE e
    WHERE salary > (
        SELECT AVG(salary) FROM EMPLOYEE WHERE dno = e.dno
    );
    ```

</div>

<div class="fragment">

??? info "**Q3 — highest-paid employee in each department**"
    ```sql
    SELECT dno, name, salary FROM EMPLOYEE e
    WHERE salary = (
        SELECT MAX(salary) FROM EMPLOYEE WHERE dno = e.dno
    );
    ```

</div>

<div class="fragment">

??? info "**Q4 — employees who work on at least one project**"
    ```sql
    SELECT name FROM EMPLOYEE e
    WHERE EXISTS (SELECT 1 FROM WORKS_ON w WHERE w.essn = e.ssn);
    ```

</div>

<div class="fragment">

??? info "**Q5 — employees who work on no projects**"
    ```sql
    SELECT name FROM EMPLOYEE e
    WHERE NOT EXISTS (SELECT 1 FROM WORKS_ON w WHERE w.essn = e.ssn);
    ```

</div>

---

## 6.11 — EXISTS, IN, ANY, ALL

???+ note "The four quantifiers"
    | Operator | Meaning |
    |---|---|
    | `IN` | Value is in the subquery result |
    | `NOT IN` | Value is not in the subquery result |
    | `EXISTS` | Subquery returns at least one row |
    | `NOT EXISTS` | Subquery returns no rows |
    | `> ANY` | Greater than at least one |
    | `> ALL` | Greater than all |

<div class="fragment">

???+ info "Example"
    ```sql
    -- Employees who earn more than any employee in dept 5
    SELECT name FROM EMPLOYEE
    WHERE salary > ANY (SELECT salary FROM EMPLOYEE WHERE dno = 5);

    -- Employees who earn more than every employee in dept 5
    SELECT name FROM EMPLOYEE
    WHERE salary > ALL (SELECT salary FROM EMPLOYEE WHERE dno = 5);
    ```

</div>

<div class="fragment">

!!! warning "NULL and NOT IN"
    `NOT IN (subquery)` returns **no rows** if the subquery returns any NULL. This is a notorious SQL trap.
    
    ```sql
    -- Broken: returns empty because dno has a NULL
    SELECT * FROM EMPLOYEE WHERE dno NOT IN (SELECT dno FROM DEPARTMENT);
    ```

</div>

---

## 6.12 — Set Operations

???+ note "Three set operators"
    | Operator | Meaning | Duplicates |
    |---|---|---|
    | `UNION` | Combine two results | Removed |
    | `UNION ALL` | Combine two results | Preserved |
    | `INTERSECT` | Rows in both | Removed |
    | `EXCEPT` / `MINUS` | Rows in first not second | Removed |

<div class="fragment">

???+ info "Example"
    ```sql
    SELECT ssn FROM EMPLOYEE
    UNION
    SELECT mgr_ssn FROM DEPARTMENT;

    SELECT ssn FROM EMPLOYEE
    INTERSECT
    SELECT essn FROM WORKS_ON;
    ```

</div>

<div class="fragment">

!!! tip "UNION ALL is faster"
    `UNION` sorts or hashes to remove duplicates; `UNION ALL` simply concatenates. Use `ALL` when duplicates are impossible or irrelevant.

</div>

---

## 6.13 — NULL Semantics

???+ note "NULL is unknown"
    NULL is not a value — it means **unknown** or **does not exist**.

<div class="fragment">

???+ info "Three-valued logic"
    Every boolean expression evaluates to **TRUE**, **FALSE**, or **UNKNOWN**:
    
    | Expression | Result |
    |---|---|
    | `NULL = NULL` | UNKNOWN |
    | `NULL > 5` | UNKNOWN |
    | `NULL AND TRUE` | UNKNOWN |
    | `NULL OR TRUE` | TRUE |
    | `NOT NULL` | UNKNOWN |

</div>

<div class="fragment">

???+ info "Testing for NULL"
    ```sql
    -- Correct
    SELECT * FROM EMPLOYEE WHERE dno IS NULL;
    SELECT * FROM EMPLOYEE WHERE dno IS NOT NULL;

    -- WRONG — always UNKNOWN (never matches)
    SELECT * FROM EMPLOYEE WHERE dno = NULL;
    ```

</div>

<div class="fragment">

!!! warning "The NOT IN trap again"
    ```sql
    -- Returns empty if any dno in DEPARTMENT is NULL
    SELECT * FROM EMPLOYEE WHERE dno NOT IN (SELECT dno FROM DEPARTMENT);
    ```
    
    **Fix:** use `NOT EXISTS` instead.

</div>

---

## 6.14 — String Operations

???+ note "LIKE pattern matching"
    | Pattern | Matches |
    |---|---|
    | `'A%'` | Starts with A |
    | `'%a'` | Ends with a |
    | `'%abc%'` | Contains abc |
    | `'_bc'` | Any char, then bc |

<div class="fragment">

???+ info "Example"
    ```sql
    SELECT name FROM EMPLOYEE WHERE name LIKE 'A%';    -- starts with A
    SELECT name FROM EMPLOYEE WHERE name LIKE '%son';  -- ends with son
    SELECT name FROM EMPLOYEE WHERE name LIKE '_ob';   -- one char + "ob"
    ```

</div>

<div class="fragment">

???+ info "Other string functions"
    ```sql
    SUBSTRING(name, 1, 3)          -- first 3 chars
    UPPER(name), LOWER(name)       -- case
    LENGTH(name)                   -- length
    TRIM(name)                     -- strip whitespace
    CONCAT(first, ' ', last)       -- concatenation
    ```

</div>

---

## 6.15 — INSERT, UPDATE, DELETE

???+ note "INSERT"
    ```sql
    -- Insert one row
    INSERT INTO EMPLOYEE (ssn, name, salary, dno)
    VALUES ('999999999', 'Alice', 50000, 5);

    -- Insert from a query
    INSERT INTO HIGH_EARNERS (ssn, name)
    SELECT ssn, name FROM EMPLOYEE WHERE salary > 100000;
    ```

<div class="fragment">

???+ note "UPDATE"
    ```sql
    UPDATE EMPLOYEE
    SET    salary = salary * 1.10
    WHERE  dno = 5;
    ```
    
    **Without WHERE, every row is updated.**

</div>

<div class="fragment">

???+ note "DELETE"
    ```sql
    DELETE FROM EMPLOYEE WHERE ssn = '999999999';
    ```
    
    **Without WHERE, every row is deleted.**

</div>

<div class="fragment">

!!! warning "Constraint violations"
    - **INSERT** with duplicate PK → `UNIQUE` violation.
    - **INSERT** with missing FK → `FOREIGN KEY` violation.
    - **DELETE** a department with employees → `FOREIGN KEY` violation (unless CASCADE).
    - **UPDATE** a PK that others reference → `FOREIGN KEY` violation (unless CASCADE).

</div>

---

## Worked example 6.3 — Data modification

!!! tip "Problem:"
    Give every employee in department 5 a 10% raise, and delete all employees in department 99.

<div class="fragment">

???+ note "**Step 1 — the raise.**"
    ```sql
    UPDATE EMPLOYEE
    SET    salary = salary * 1.10
    WHERE  dno = 5;
    ```
    
    The `WHERE` is **essential** — without it, all employees get the raise.

</div>

<div class="fragment">

??? info "**Step 2 — the deletion.**"
    ```sql
    DELETE FROM EMPLOYEE WHERE dno = 99;
    ```
    
    If `EMPLOYEE` has foreign keys to `WORKS_ON` or `DEPENDENT` with `RESTRICT`, this fails — you must first delete dependent rows (or use `ON DELETE CASCADE`).

</div>

<div class="fragment">

??? success "**Step 3 — transaction safety.**"
    ```sql
    BEGIN;
    UPDATE EMPLOYEE SET salary = salary * 1.10 WHERE dno = 5;
    DELETE FROM EMPLOYEE WHERE dno = 99;
    COMMIT;
    ```
    
    Wrap multi-statement operations in a transaction so they either all succeed or all roll back.

</div>

---

## 6.16 — Views

???+ note "Definition"
    A **view** is a named, stored query that behaves like a table.

<div class="fragment">

???+ info "Example"
    ```sql
    CREATE VIEW HIGH_EARNERS AS
    SELECT ssn, name, salary
    FROM   EMPLOYEE
    WHERE  salary > 100000;

    SELECT * FROM HIGH_EARNERS;  -- query the view
    ```

</div>

<div class="fragment">

???+ info "Why views?"
    - **Security** — expose only a subset of the data.
    - **Simplicity** — encapsulate complex joins.
    - **Logical data independence** — change the underlying schema without breaking apps that use the view.

</div>

<div class="fragment">

!!! warning "Updatable views"
    Simple views (no aggregates, no joins, no DISTINCT) are **updatable** — you can INSERT/UPDATE through them. Complex views are **read-only**.

</div>

---

## 6.17 — Indexes and Query Plans

???+ note "What an index does"
    An **index** is a data structure (usually a B-tree) that speeds up lookups on a column.

<div class="fragment">

???+ info "Creating an index"
    ```sql
    CREATE INDEX idx_employee_salary ON EMPLOYEE(salary);
    CREATE INDEX idx_employee_dno    ON EMPLOYEE(dno);
    ```

</div>

<div class="fragment">

???+ info "The EXPLAIN command"
    ```sql
    EXPLAIN ANALYZE
    SELECT name FROM EMPLOYEE WHERE salary > 50000;
    ```
    
    Shows the query plan (index scan vs sequential scan) and the actual timing.

</div>

<div class="fragment">

!!! tip "When to add an index"
    - Frequently filtered columns (`WHERE`).
    - Frequently joined columns (foreign keys).
    - Frequently sorted columns (`ORDER BY`).
    
    But **every index slows down writes** — balance carefully.

</div>

---

## Chapter 6 — key takeaways

<div class="fragment">

**1.** **DDL** creates/modifies the schema: `CREATE`, `ALTER`, `DROP` with constraints (`PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `NOT NULL`, `CHECK`).

</div>

<div class="fragment">

**2.** **SELECT** evaluation order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY`. Aggregates cannot appear in `WHERE`.

</div>

<div class="fragment">

**3.** **Joins**: `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, and `NATURAL`. Forgetting the join condition produces a Cartesian product.

</div>

<div class="fragment">

**4.** **Aggregates** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) with `GROUP BY` and `HAVING` — the pattern for summary queries.

</div>

<div class="fragment">

**5.** **NULL** semantics: three-valued logic; `NULL = NULL` is UNKNOWN; use `IS NULL`. Watch for the `NOT IN` + `NULL` trap.

</div>

<div class="fragment">

**6.** **INSERT**, **UPDATE**, **DELETE** respect constraints and referential actions. Wrap multi-statement operations in transactions.

</div>

---

## Next — Chapter 7

**Disk Storage, File Structures, and Hashing.**

<div class="fragment">

- Disk structure and access time.
- Files of records; slotted pages.
- **Heap**, **sorted**, **hashed** file organizations.
- **Static**, **dynamic**, **extendible** hashing.
- **RAID** levels 0–6.

</div>

<div class="fragment">

Chapters 1–6 described the **logical** and **query** levels.