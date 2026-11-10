# Schedule

Week-by-week plan for **ECEG-4191**, 18-week semester.

---

## At a glance

| Week | Chapter | Topic | Tutorial | Lab | Assessment |
|:---:|:---:|---|---|---|---|
| 1 | 1 | Introduction — file systems, DBMS | — | — | — |
| 2 | 1 | ANSI-SPARC architecture, DBLC | T1 | L1 | A1 out |
| 3 | 2 | Relational model — relations, keys | — | — | — |
| 4 | 2 | Integrity constraints, schema vs. instance | T2 | L1 | A1 due / A2 out |
| 5 | 3 | ER model — entities, attributes, relationships | — | — | — |
| 6 | 3 | ER structural constraints; weak entities; EER | T3 | L1 | A2 due / A3 out |
| 7 | 4 | ER-to-relational mapping (steps 1–7) | — | — | A3 due |
| 8 | — | **Mid-semester test** | — | — | **15%** |
| 9 | 4 | EER-to-relational mapping; design guidelines | T4 | L2 | A4 out |
| 10 | 4 | FDs, closure, minimal cover | — | — | A4 due / A5 out |
| 11 | 4 | Normalisation 1NF, 2NF, 3NF, BCNF | T5 | L5 | — |
| 12 | 5 | Relational algebra — operations, queries | — | — | A5 due / A6 out |
| 13 | 5 | Tuple and domain calculus | T6 | L2 | — |
| 14 | 6 | SQL — DDL, basic queries | — | — | A6 due / A7 out |
| 15 | 6 | SQL — joins, aggregates, subqueries | T7 | L3 | Test 2 **15%** |
| 16 | 7 | Storage — files, hashing, RAID | T8 | L4 | A7 due / A8 out |
| 17 | — | Review and project demos | — | — | A8 due / Project due |
| 18 | — | **Final examination** | — | — | **40%** |

**Legend.** T = tutorial · L = laboratory · A = assignment · — = no activity

---

## Detail by week

### Week 1 — Chapter 1, Part 1

- **Lecture.** File-based systems and their limitations; the database approach; DBMS functions and components.
- **Reading.** Elmasri §1.1–§1.4; Silberschatz §1.1–§1.3.

### Week 2 — Chapter 1, Part 2

- **Lecture.** ANSI-SPARC three-schema architecture; data independence; multi-user DBMS; database development life cycle.
- **Tutorial 1.** File-based vs. database comparison; schema/instance exercises.
- **Laboratory 1.** ER diagramming with draw.io.
- **Assignment 1** released.

### Week 3 — Chapter 2, Part 1

- **Lecture.** The relational model; relations, tuples, attributes, domains; keys (superkey, candidate, primary, foreign).
- **Reading.** Elmasri §3.1–§3.3; Silberschatz §2.1–§2.3.

### Week 4 — Chapter 2, Part 2

- **Lecture.** Entity and referential integrity constraints; schema vs. instance; relational schema notation.
- **Tutorial 2.** Key identification exercises; constraint violation detection.
- **Assignment 1 due; Assignment 2** released.

### Week 5 — Chapter 3, Part 1

- **Lecture.** The ER model — entities, attributes, relationships; cardinality ratios (1:1, 1:N, M:N); participation (total, partial).
- **Reading.** Elmasri §3.4–§3.6.

### Week 6 — Chapter 3, Part 2

- **Lecture.** Weak entities; EER concepts: subclasses, superclasses, specialisation, generalisation, categories; attribute and relationship inheritance.
- **Tutorial 3.** Draw ER/EER diagrams for the COMPANY case study.
- **Assignment 2 due; Assignment 3** released.

### Week 7 — Chapter 4, Part 1

- **Lecture.** ER-to-relational mapping steps 1–7 (regular entities, weak entities, 1:1, 1:N, M:N, multivalued attributes, n-ary relationships).
- **Assignment 3 due.**

### Week 8 — Mid-semester test

- **Test.** 2 hours, closed book, **15%** of final grade. Covers Chapters 1–3.

### Week 9 — Chapter 4, Part 2

- **Lecture.** EER-to-relational mapping (steps 8–9); informal design guidelines (semantic information, redundancy, NULL values, spurious tuples).
- **Tutorial 4.** Full ER-to-relational mapping on the COMPANY database.
- **Laboratory 2.** Implement the mapped schema as `CREATE TABLE` statements.
- **Assignment 4** released.

### Week 10 — Chapter 4, Part 3

- **Lecture.** Functional dependencies; Armstrong's axioms; closure; equivalence; minimal cover.
- **Reading.** Elmasri §14.1–§14.3.
- **Assignment 4 due; Assignment 5** released.

### Week 11 — Chapter 4, Part 4

- **Lecture.** 1NF, 2NF, 3NF, BCNF; decomposition properties (lossless join, dependency preservation).
- **Tutorial 5.** Normalisation exercises on small schemas.
- **Assignment 5 due; Assignment 6** released.

### Week 12 — Chapter 5, Part 1

- **Lecture.** Relational algebra: σ, π, ×, natural join, θ-join, ∪, ∩, −, division; query examples.
- **Reading.** Elmasri §6.1–§6.3.

### Week 13 — Chapter 5, Part 2

- **Lecture.** Tuple and domain relational calculus; safe expressions; equivalence of RA, TRC, DRC.
- **Tutorial 6.** Translate SQL-like queries into RA, TRC, DRC.
- **Assignment 6 due; Assignment 7** released.

### Week 14 — Chapter 6, Part 1

- **Lecture.** SQL DDL — CREATE DATABASE, CREATE TABLE, ALTER, DROP; constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK); referential-integrity options (CASCADE, SET NULL, RESTRICT).
- **Reading.** Elmasri §4.1–§4.2; Silberschatz §3.1–§3.2.

### Week 15 — Chapter 6, Part 2

- **Lecture.** SQL DML — SELECT (basic, WHERE, aliases, DISTINCT, ORDER BY); joins (INNER, LEFT, RIGHT); aggregates and GROUP BY; HAVING; subqueries; EXISTS; UNION/INTERSECT/MINUS; NULL handling; INSERT, UPDATE, DELETE.
- **Tutorial 7.** Query-writing exercises on the COMPANY database.
- **Laboratory 3.** SQL queries on the sample database.
- **Test 2** (week 15, **15%**) — Chapters 4–5.
- **Assignment 7 due; Assignment 8** released.

### Week 16 — Chapter 7

- **Lecture.** Disk storage devices; files of records; unordered, sorted, and hashed files; static, dynamic, and extendible hashing; RAID 0–6.
- **Tutorial 8.** Compute disk I/O costs for heap vs. sorted vs. hashed access.
- **Laboratory 4.** Benchmark B-tree vs. hash index lookup times.
- **Assignment 8 due; Project due.**

### Week 17 — Review

- **Lecture.** Review of all seven chapters; worked exam-style problems.
- **Tutorial.** Project demonstrations and Q&A.
- **Project due.**

### Week 18 — Final examination

- **Exam.** 3 hours, closed book, **40%** of final grade. Covers the whole syllabus.