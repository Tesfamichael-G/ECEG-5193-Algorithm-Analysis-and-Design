# Exam Bank — ECEG-4191 Database Systems

**40 questions spanning all seven chapters.** Use for self-assessment and exam preparation.

**Instructions (mock exam):**

- Time: **3 hours**
- Closed book; **one A4 handwritten formula sheet** allowed
- Total marks: **100**
- Marks per question in brackets

---

## Section A — Introduction (Ch 1) — 10 marks

### A1. Multiple choice [2]

Which of the following is **not** a limitation of file-based systems?

(a) data redundancy
(b) data inconsistency
(c) program-data dependence
(d) support for concurrent access by multiple users

### A2. Multiple choice [2]

In the ANSI-SPARC three-schema architecture, the schema that describes the **physical storage structure** is the:

(a) external schema
(b) conceptual schema
(c) internal schema
(d) view schema

### A3. Short answer [3]

List three advantages of the database approach over file-based systems. For each, give one concrete example.

### A4. Short answer [3]

Explain the difference between **logical** and **physical** data independence. Why is physical data independence easier to achieve than logical?

---

## Section B — Relational Model (Ch 2) — 10 marks

### B1. Multiple choice [2]

In a relation, a **candidate key** is:

(a) any attribute
(b) a minimal superkey
(c) a foreign key
(d) the first attribute

### B2. Multiple choice [2]

The **referential integrity constraint** states that:

(a) every tuple has a unique primary key
(b) a foreign key value must either be NULL or match a primary key value in the referenced relation
(c) no attribute can contain NULLs
(d) every relation must have at least one tuple

### B3. Short answer [3]

Given a relation `EMPLOYEE(emp_id, name, dept_id)` and `DEPARTMENT(dept_id, dept_name)`, identify a candidate key, a primary key, and a foreign key. Explain each choice.

### B4. Short answer [3]

Distinguish between a **relation schema** and a **relation instance**. Give an example of each for the `EMPLOYEE` relation.

---

## Section C — ER Modelling (Ch 3) — 15 marks

### C1. Multiple choice [2]

A **weak entity** is one that:

(a) has no attributes
(b) cannot exist without a related owner entity
(c) has only one attribute
(d) has no primary key at all

### C2. Multiple choice [2]

In the EER model, **generalisation** is the process of:

(a) splitting a superclass into subclasses
(b) combining subclasses into a superclass
(c) adding more attributes to an entity
(d) removing relationships

### C3. Short answer [4]

For the COMPANY database described below, draw a complete ER diagram:

- **EMPLOYEE** (ssn, name, address, salary, sex, birth_date)
- **DEPARTMENT** (dnumber, dname)
- **PROJECT** (pnumber, pname, location)
- Each department has exactly one manager (an employee) and controls many projects.
- Each employee works in exactly one department and may work on many projects with a weekly `hours` attribute.
- Each employee may have many dependents, each with `name`, `sex`, `birth_date`, and `relationship`.

State every cardinality ratio, participation constraint, and any weak entity.

### C4. Short answer [4]

Given an entity **PERSON** with subclasses **STUDENT** and **EMPLOYEE**, both of which can further have subclasses **GRAD_STUDENT** and **STAFF**:

- Draw the EER diagram with specialisation/generalisation.
- State whether the specialisation is **disjoint** or **overlapping** — justify your choice.
- State whether it is **total** or **partial** — justify your choice.

### C5. Short answer [3]

Explain **attribute inheritance** in the EER model. Give one example where inheritance is useful and one where it would be inappropriate.

---

## Section D — Logical Design and Normalisation (Ch 4) — 25 marks

### D1. Multiple choice [2]

The **third normal form (3NF)** eliminates:

(a) partial dependencies on a candidate key
(b) transitive dependencies on a candidate key
(c) all functional dependencies
(d) the need for a primary key

### D2. Multiple choice [2]

A relation is in **BCNF** if, for every non-trivial functional dependency $X \to Y$:

(a) $Y$ is a candidate key
(b) $X$ is a superkey
(c) $X \subseteq Y$
(d) $Y \subseteq X$

### D3. Short answer [4]

Map the following ER model to relational schemas (show all steps 1–7):

- **STUDENT** (student_id, name, address — multivalued)
- **COURSE** (course_id, title)
- **ENROLLMENT** — M:N relationship between STUDENT and COURSE, with attribute `grade`
- **DEPENDENT** — weak entity of STUDENT with attribute `dep_name`
- **PROFESSOR** (prof_id, name) — each professor advises many students (1:N relationship with STUDENT)

Write the resulting `CREATE TABLE` statements (keys, foreign keys, and constraints).

### D4. Derivation [5]

Let $R(A, B, C, D, E)$ with functional dependencies:

$$
AB \to C, \quad C \to D, \quad D \to E.
$$

1. Compute the **closure of** $AB$.
2. Identify all **candidate keys** of $R$.
3. Decompose $R$ into **3NF** using the synthesis algorithm.
4. Is the decomposition also in **BCNF**? Justify.

### D5. Short answer [4]

Given $R(A, B, C, D)$ with $AB \to C$, $C \to D$, and $D \to A$:

- Is $R$ in 3NF? In BCNF? Justify.
- If not in BCNF, decompose it and verify that the decomposition is lossless.

### D6. Short answer [3]

Explain the three informal design guidelines for relational databases:

1. **Semantics of attributes** — why should each tuple represent one entity?
2. **Redundant information** — what anomalies does it cause?
3. **NULL values** — what problems do they introduce in joins?

Give one example of each.

### D7. Short answer [5]

Consider the relation:

`EMPLOYEE_PROJECT(emp_id, emp_name, dept_name, proj_id, proj_name, hours)`

where each employee works on one or more projects, `emp_name` and `dept_name` depend only on `emp_id`, and `proj_name` depends only on `proj_id`.

1. List the functional dependencies.
2. Identify all candidate keys.
3. Normalise to **3NF**.
4. State one update anomaly that the original schema suffers from.

---

## Section E — Relational Algebra and Calculus (Ch 5) — 15 marks

### E1. Multiple choice [2]

The **division** operation $R \div S$ in relational algebra returns:

(a) the Cartesian product of $R$ and $S$
(b) tuples of $R$ that match **all** tuples of $S$ on the common attributes
(c) tuples of $R$ that match **some** tuples of $S$
(d) the union of $R$ and $S$

### E2. Short answer [4]

Using the COMPANY database from Section C:

- **Dnumber of the department with the highest-paid manager.**
- **Names of employees who work on projects in more than 2 departments.**
- **Names of employees who work on all projects.**

Express each query in **relational algebra**. You may assume the relations `EMPLOYEE(ssn, name, salary, dno)`, `DEPARTMENT(dnumber, mgr_ssn)`, `PROJECT(pnumber, dnum)`, `WORKS_ON(essn, pno)`.

### E3. Short answer [4]

Express the following query in **both** relational algebra **and** tuple relational calculus:

> "Find the names of departments in which **every** employee earns more than 30,000."

### E4. Short answer [5]

Given relations $R(A, B)$ and $S(B, C)$, write queries that return:

1. The **natural join** $R \bowtie S$.
2. The **left outer join** $R ⟕ S$.
3. Pairs $(a, c)$ such that $B$ in $R$ is **not** in $S$.
4. All $A$ values that appear in $R$ with **at least two different** $B$ values.

Express each in relational algebra.

---

## Section F — SQL (Ch 6) — 15 marks

### F1. Multiple choice [2]

Which SQL clause is evaluated **last** (i.e., after GROUP BY and HAVING)?

(a) SELECT
(b) WHERE
(c) ORDER BY
(d) FROM

### F2. Multiple choice [2]

The difference between `WHERE` and `HAVING` is that:

(a) `WHERE` filters rows before grouping, `HAVING` filters groups after grouping
(b) `HAVING` filters rows before grouping, `WHERE` filters groups after grouping
(c) they are equivalent
(d) `WHERE` only works with aggregates

### F3. Short answer [4]

Using the COMPANY database, write the SQL for each query:

1. Names of employees whose salary is above the department average.
2. Names of departments with more than 5 employees and an average salary of at least 40,000.
3. Names of employees who **do not** work on any project.
4. Names of employees who work on **all** projects.

### F4. Short answer [4]

Write the DDL to create the `EMPLOYEE` and `DEPARTMENT` tables from the COMPANY database:

- `EMPLOYEE(ssn, name, salary, dno)` — `dno` references `DEPARTMENT`
- `DEPARTMENT(dnumber, dname, mgr_ssn)` — `mgr_ssn` references `EMPLOYEE`

Include `PRIMARY KEY`, `FOREIGN KEY`, `NOT NULL`, and `UNIQUE` constraints. Handle the circular reference appropriately.

### F5. Short answer [3]

Explain the difference between:

- `EXISTS` and `IN`.
- `UNION` and `UNION ALL`.

Give one query where each choice matters.

---

## Section G — Storage and Hashing (Ch 7) — 10 marks

### G1. Multiple choice [2]

**RAID level 1** is:

(a) striping without redundancy
(b) mirroring
(c) bit-level striping with parity
(d) block-level striping with distributed parity

### G2. Multiple choice [2]

**Extendible hashing** differs from **static hashing** in that it:

(a) stores all records in one bucket
(b) grows and shrinks the directory dynamically as records are added or removed
(c) uses a linked list of overflow blocks
(d) requires a pre-allocated file of fixed size

### G3. Short answer [3]

Compare **static hashing**, **dynamic hashing**, and **extendible hashing** on:

- overflow handling,
- directory size,
- growth behaviour.

### G4. Short answer [3]

Explain the **RAID 0, 1, 5** levels:

- How does each distribute data across disks?
- What is the trade-off between performance, redundancy, and cost?

---

## Grading rubric

| Band | Marks |
|---|---|
| A | 90–100 |
| B | 75–89 |
| C | 60–74 |
| F | < 60 |

**Common error checks:**

- Missing foreign key in a mapping exercise: −2.
- Mapping a multivalued attribute into the same relation instead of a separate table: −3.
- Confusing 3NF with BCNF: −2.
- Natural join used where a theta join is needed: −2.
- Aggregate function used in `WHERE` instead of `HAVING`: −2.
- Missing `NOT NULL` on a primary key in DDL: −1.
- Confusing RAID 1 with RAID 5: −2.

---

## Answer key

Full worked solutions are in the [Solutions](../solutions/index.md) section — one file per chapter, each answering the corresponding section above.