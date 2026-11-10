# Syllabus

Full course outline for **ECEG-4191 — Database Systems**.

---

## Course identification

| Field | Value |
|---|---|
| **Course code** | ECEG-4191 |
| **Course title** | Database Systems |
| **Credit hours (ECTS)** | 5 — 2 lecture + 1 tutorial + 2 lab per week |
| **Degree program** | BSc Electrical Engineering (Computer) |
| **Module** | Data Storage and Analysis |
| **Module coordinator** | N.N. |
| **Semester** | VII/7 |
| **Status** | Compulsory |
| **Pre-requisites** | Object-Oriented Programming (co- or pre-requisite) |
| **Instructor** | *(insert)* |

---

## Student workload

| Lecture | Tutorial | Practice / Laboratory | Home study |
|:---:|:---:|:---:|:---:|
| 2 h | 1 h | 2 h | ~5 h |

Total: ~10 hours per week × 18 weeks = **~180 hours** — consistent with 5 ECTS.

---

## Course description

This course provides a solid foundation in database systems — both the theoretical principles and the practical skills needed to design, implement, and query a relational database. It follows the canonical lifecycle: from understanding requirements, through conceptual and logical design, to physical design and SQL implementation. It closes with the storage-level concerns of disk files and hashing.

---

## Course objectives and competences

Upon completion of this course, students will have gained knowledge of database system concepts and the ability to:

1. **Understand** user requirements and views.
2. **Analyse** existing and future data-processing needs.
3. **Develop** an enterprise data model that reflects the organisation's fundamental business rules.
4. **Develop and refine** the conceptual data model, including all entities, relationships, attributes, and business rules.
5. **Integrate and merge** database views into a conceptual model.
6. **Apply** normalisation techniques.
7. **Derive** a physical design from the logical design taking into account application, hardware, operating system, and data communications network requirements.

---

## Learning outcomes

| # | Outcome | Assessed by |
|---|---|---|
| LO1 | Explain the database approach, DBMS components, and ANSI-SPARC architecture | Project, Test 1 |
| LO2 | Define relations, keys, and integrity constraints of the relational model | Test 1 |
| LO3 | Draw ER and EER diagrams for real-world requirements | Project, Test 1 |
| LO4 | Map ER/EER models to relational schemas; apply design guidelines | Project, Test 2 |
| LO5 | Compute closures, minimal covers, and normalise to 3NF and BCNF | Test 2, Final |
| LO6 | Express queries in relational algebra and in SQL | Lab, Final |
| LO7 | Design file structures and choose hashing/RAID schemes | Final |

---

## Course content

**Chapter 1 — Introduction**

- Manual file handling systems and limitations
- Traditional file-based systems
- Database approach
- Database management system (DBMS)
- DBMS and components of the DBMS environment
- Database development life cycle
- Common uses of databases
- ANSI-SPARC architecture
- Multi-user DBMS

**Chapter 2 — The Relational Model**

- Data models
- Relational data model
- Relational database
- Database terminology
- Relational constraints
- Schema and instances

**Chapter 3 — Conceptual Database Design, E-R Modelling**

- Database design phases
- Conceptual database design
- Basic concepts of the E-R model
- Graphical representation of E-R diagrams
  - Structural constraints on relationships
  - Participation constraints
  - EER concepts: subclasses/superclasses, specialisation/generalisation, categories (UNION types)
  - Attribute and relationship inheritance
- Case study: COMPANY database

**Chapter 4 — Logical Database Design**

- ER-to-Relational mapping algorithms (7 steps)
- EER-to-Relational mapping algorithms (steps 8 and 9)
- Informal design guidelines for relational databases
- Functional dependencies (definition, inference rules, equivalence, minimal cover)
- Normal forms: 1NF, 2NF, 3NF, general NF definitions, BCNF
- Practical use of normal forms

**Chapter 5 — Relational Algebra and Calculus**

- Relational algebra: unary, set-theoretic, binary operations, joins, division
- Tuple relational calculus (TRC)
- Domain relational calculus (DRC)
- Examples: the COMPANY database

**Chapter 6 — Structured Query Language (SQL)**

- Objectives of SQL
- DDL: CREATE, ALTER, DROP; referential-integrity options
- DML: SELECT, PROJECT, JOIN; aliases; DISTINCT; UNION/INTERSECT/MINUS; subqueries; aggregate functions; GROUP BY; HAVING; ORDER BY
- Data modification: INSERT, DELETE, UPDATE
- EXISTS; explicit sets; NULLs; SUBSTRING

**Chapter 7 — Disk Storage, File Structures, and Hashing**

- Disk storage devices
- Files of records and operations on files
- Unordered files; ordered files; hashed files
- Dynamic and extendible hashing
- RAID technology

---

## Laboratory exercises

| Lab | Topic |
|---|---|
| L1 | ER diagramming with a CASE tool (draw.io, Lucidchart, or DIA) |
| L2 | ER-to-relational mapping and DDL (CREATE TABLE with keys and constraints) |
| L3 | Simple SQL queries on a sample COMPANY database |
| L4 | Complex SQL: joins, GROUP BY, HAVING, subqueries |
| L5 | Normalisation exercises: compute closure, minimal cover, BCNF decomposition |
| L6 | Index and file-structure experiment (B-tree vs. hash index timing) |

---

## Project (30% of grade)

A semester-long project in which students:

1. Design a database for a given application domain.
2. Produce an ER/EER diagram.
3. Map it to a relational schema.
4. Normalise the schema.
5. Implement it in a real DBMS (PostgreSQL or MySQL).
6. Write SQL queries and load sample data.
7. Deliver a final report and a demonstration.

---

## Assessment

| Component | Weight |
|---|:---:|
| Project and Assignment | 30% |
| Tests | 30% |
| Final examination | 40% |
| **Total** | **100%** |

---

## Textbook and references

**Textbook.**

- **Elmasri, R. & Navathe, S. B.**, *Fundamentals of Database Systems*, 7th ed., Pearson, 2016.

**References.**

- **Silberschatz, A., Korth, H. F. & Sudarshan, S.**, *Database System Concepts*, 6th ed., McGraw-Hill, 2010.
- **Ramakrishnan, R. & Gehrke, J.**, *Database Management Systems*, 3rd ed., McGraw-Hill, 2002.
- **Garcia-Molina, H., Ullman, J. D. & Widom, J.**, *Database Systems: The Complete Book*, 2nd ed., Pearson, 2008.
- **Pervasive Software Inc.**, *Database Design Guide*.

---

## Policies

- **Attendance:** minimum **75%** required.
- **Late work:** 10% per day, zero after 3 days.
- **Academic integrity:** all submitted work must be your own.