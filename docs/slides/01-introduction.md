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
  js: assets/lecture-01.js

---

# ECEG-4191

## Chapter 1 — Introduction to Database Systems

---

## Learning outcomes

By the end of this chapter you should be able to:

<div class="fragment">

1. Explain **why file-based systems fail** at scale, with five concrete limitations.

</div>

<div class="fragment">

2. Define a **DBMS** and describe its five components of the environment.

</div>

<div class="fragment">

3. Draw and explain the **ANSI-SPARC three-schema architecture** and the two types of data independence.

</div>

<div class="fragment">

4. Describe the **database development life cycle** (DBLC) and the role of each phase.

</div>

<div class="fragment">

5. Identify the **five components** of a multi-user DBMS and the trade-offs between them.

</div>

---

## 1.1 — File-Based Systems

???+ note "Definition"
    A **file-based system** stores data in separate files, each managed by its own application program. There is no centralized control over the data.

<div class="fragment">

???+ info "The canonical structure"
    ```
    Application 1  ───►  File 1 (Students)
    Application 2  ───►  File 2 (Courses)
    Application 3  ───►  File 3 (Enrollments)
    ```
    
    Each application has its own view of the data — and its own copy.

</div>

<div class="fragment">

!!! warning "The fundamental problem"
    Data is **owned by applications**, not by the enterprise. Any change to the data format requires updating every application that touches it.

</div>

---

## 1.2 — Five Limitations of File-Based Systems

???+ note "1. Data redundancy"
    The same fact is stored in multiple files. A student's name appears in `Students`, `Enrollments`, and `Grades`.

<div class="fragment">

???+ note "2. Data inconsistency"
    Different copies diverge over time. Updating the name in one file but not the others creates **contradictions**.

</div>

<div class="fragment">

???+ note "3. Program-data dependence"
    The file format is defined **inside the program**. Changing the format (adding a column, changing a length) requires editing every program.

</div>

<div class="fragment">

???+ note "4. Difficulty of access"
    New questions ("Which students took CS301 in 2023?") require writing **new programs**, not just new queries.

</div>

<div class="fragment">

???+ note "5. Concurrency and atomicity"
    Multiple users can corrupt data. A crash mid-update leaves the data **inconsistent**.

</div>

---

## 1.2 — Concrete Example of Redundancy

???+ info "A university with three applications"
    | Application | File | Fields |
    |---|---|---|
    | Registrar | `Students.dat` | id, name, address, dept |
    | Payroll | `Payroll.dat` | id, name, dept, salary |
    | Library | `Loans.dat` | id, name, book, date |

<div class="fragment">

!!! warning "Three copies of `name` and `dept`"
    - **Update anomaly:** when a student changes their name, three files must be updated — if any is missed, the data becomes inconsistent.
    - **Insertion anomaly:** you cannot record a new department until a student joins it.
    - **Deletion anomaly:** deleting a student's payroll record also deletes their `name` and `dept` — even if the registrar still needs them.

</div>

<aside class="notes">
  <div style="font-size: 16px; line-height: 1.6;">
These three anomalies — update, insertion, deletion — are the concrete symptoms of redundancy. The database approach does not just "store data in one place"; it eliminates the structural causes of these anomalies. That is the real argument for normalization (Chapter 4).
  </div>
</aside>

---

## 1.3 — The Database Approach

???+ note "Definition (Elmasri §1.3)"
    A **database** is an organized collection of structured data. A **DBMS** is software that manages and controls access to the database.

<div class="fragment">

???+ info "Key properties of the database approach"
    | Property | Meaning |
    |---|---|
    | **Self-describing** | The database stores its own schema (catalog / data dictionary) |
    | **Insulation** | Programs are independent of data format |
    | **Multiple views** | Different users see different subsets |
    | **Sharing** | Multiple users and programs share one store |
    | **Transaction support** | ACID properties |
    | **Integrity constraints** | Enforced by the DBMS, not the program |

</div>

<div class="fragment">

!!! tip "The key insight"
    Data is a **shared corporate resource**. Applications come and go; data persists. The DBMS is the layer that lets data outlive applications.

</div>

---

## 1.4 — DBMS Environment: Five Components

???+ note "Five components (Elmasri §1.5)"
    ```
    ┌──────────────────────────────────────────┐
    │  People: users, programmers, DBAs,       │
    │          designers                       │
    ├──────────────────────────────────────────┤
    │  Procedures: instructions for using      │
    │              the system                  │
    ├──────────────────────────────────────────┤
    │  Software: DBMS + application programs   │
    ├──────────────────────────────────────────┤
    │  Data: the database + the catalog        │
    ├──────────────────────────────────────────┤
    │  Hardware: storage, CPU, network         │
    └──────────────────────────────────────────┘
    ```

<div class="fragment">

!!! warning "The people component is not an afterthought"
    The DBA (Database Administrator) is responsible for:
    - **Schema design** and evolution.
    - **Security** (who can access what).
    - **Performance tuning** (indexes, statistics).
    - **Backup and recovery**.
    - **Capacity planning**.

</div>

---

## 1.5 — Functions of a DBMS

???+ note "What the DBMS actually does"
    | Function | Description |
    |---|---|
    | **Data storage** | Persistent, structured, at scale |
    | **Data retrieval** | Query processing and optimization |
    | **Transaction management** | ACID: atomicity, consistency, isolation, durability |
    | **Concurrency control** | Multiple users without corruption |
    | **Recovery** | Restore a consistent state after a crash |
    | **Security** | Authentication and authorization |
    | **Integrity** | Enforce constraints (keys, references) |
    | **Catalog** | Metadata about the schema |

<div class="fragment">

???+ note "ACID in one line each"
    - **Atomicity:** a transaction is all-or-nothing.
    - **Consistency:** a transaction moves the database from one valid state to another.
    - **Isolation:** concurrent transactions do not see each other's intermediate states.
    - **Durability:** a committed transaction survives a crash.

</div>

---

## 1.6 — ANSI-SPARC Three-Schema Architecture

???+ note "The three levels (Elmasri §2.2)"
    ```
    External schema 1     External schema 2     External schema 3
         (view)                 (view)                (view)
               \                  |                  /
                \                 |                 /
                 ─────►  Conceptual schema  ◄─────
                          (community view)
                              |
                              |
                              ▼
                         Internal schema
                        (storage details)
                              |
                              ▼
                         Stored database
    ```

<div class="fragment">

???+ note "The three schemas"
    - **External (view) level** — what each user sees.
    - **Conceptual (logical) level** — the enterprise-wide logical structure.
    - **Internal (physical) level** — how data is stored.

</div>

<aside class="notes">
  <div style="font-size: 16px; line-height: 1.6;">
The three-schema architecture is the single most important design idea in databases. It is what makes data independence possible. Every modern DBMS implements it, even though the internal schema is often hidden from users.
  </div>
</aside>

---

## 1.7 — Data Independence

???+ abstract "Logical data independence"
    The ability to change the **conceptual schema** without changing the **external schemas**.
    
    *Example:* add a `phone_number` column to `EMPLOYEE` — applications that don't select it are unaffected.

<div class="fragment">

???+ abstract "Physical data independence"
    The ability to change the **internal schema** without changing the **conceptual schema**.
    
    *Example:* change an index from B-tree to hash — queries written against the conceptual schema still work.

</div>

<div class="fragment">

!!! tip "Which is harder?"
    **Logical data independence is harder** to achieve because changes to the conceptual schema can affect queries in subtle ways. Physical data independence is "automatic" — the query processor chooses the physical access path.

</div>

---

## 1.8 — Database Languages

???+ note "The three languages of a DBMS"
    | Language | Purpose | Example statement |
    |---|---|---|
    | **DDL** | Define and modify the schema | `CREATE TABLE ...` |
    | **DML** | Query and modify the data | `SELECT ...`, `INSERT ...` |
    | **DCL** | Control access | `GRANT SELECT ON ... TO alice` |

<div class="fragment">

???+ info "DDL is compiled, DML is interpreted"
    - DDL statements store **metadata** in the catalog.
    - DML statements are **compiled** into a query plan, then **executed**.
    - This is why the query processor has two stages: compilation and runtime.

</div>

---

## 1.9 — Database Development Life Cycle

???+ note "Eight phases"
    | Phase | Output |
    |---|---|
    | **1. Requirements analysis** | Requirements specification |
    | **2. Conceptual design** | ER / EER model |
    | **3. Choice of DBMS** | Vendor/product decision |
    | **4. Logical design** | Relational schema, normalized |
    | **5. Physical design** | Indexes, file structures, partitioning |
    | **6. Implementation** | Schema in the DBMS, data loaded |
    | **7. Testing** | Validated against requirements |
    | **8. Operation and maintenance** | Monitoring, tuning, evolution |

<div class="fragment">

!!! warning "The DBLC is iterative"
    Design flaws discovered in later phases force refinement of earlier ones. Most real projects iterate 2–3 times through the design phases.

</div>

---

## 1.10 — Requirements Analysis

???+ note "What goes into requirements"
    - **Data requirements** — what entities, attributes, relationships.
    - **Functional requirements** — what operations (queries, updates).
    - **Non-functional requirements** — performance, security, availability.

<div class="fragment">

???+ info "Techniques"
    - **Interviews** with stakeholders.
    - **Observation** of existing workflows.
    - **Analysis of existing documents** (forms, reports).
    - **Prototyping** — mock-ups to clarify requirements.

</div>

<div class="fragment">

!!! tip "The cost of bad requirements"
    Errors in requirements are the **most expensive to fix** — they propagate through every downstream phase. A 10× cost multiplier per phase is typical (Boehm's curve).

</div>

---

## 1.11 — Multi-User DBMS

???+ note "What changes when there are many users?"
    | Concern | Why it matters |
    |---|---|
    | **Concurrency control** | Simultaneous updates can corrupt data |
    | **Transaction management** | Commit/rollback on failure |
    | **Recovery** | Restore consistency after a crash |
    | **Security** | Authenticate, authorize |
    | **Views** | Present tailored subsets |
    | **Performance** | Scale with user count |

<div class="fragment">

???+ note "The concurrency problem in one line"
    Two transactions reading and writing the same data can produce a state **no serial execution would produce** — a *race condition* at the database level.

</div>

<div class="fragment">

!!! tip "Isolation levels"
    SQL standard defines four isolation levels (READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE). Higher isolation = fewer anomalies but less concurrency. This is the classic trade-off in multi-user DBMS.

</div>

---

## 1.12 — Common Uses of Database Systems

???+ info "Where databases appear"
    | Domain | Example system |
    |---|---|
    | **Banking** | Account balances, transactions, loans |
    | **Airlines** | Reservations, schedules, crew |
    | **Universities** | Students, courses, grades, faculty |
    | **E-commerce** | Products, orders, inventory, customers |
    | **Healthcare** | Patients, records, prescriptions |
    | **Telecommunications** | Call records, subscriptions, billing |
    | **Social networks** | Users, posts, friendships, likes |

<div class="fragment">

!!! tip "Two patterns"
    - **OLTP (Online Transaction Processing)** — many small, fast transactions. Banking, e-commerce.
    - **OLAP (Online Analytical Processing)** — few large, complex queries. Data warehouses, business intelligence.

</div>

---

## Chapter 1 — key takeaways

<div class="fragment">

**1.** File-based systems fail because of **redundancy, inconsistency, program-data dependence, difficulty of access, and concurrency problems** — leading to update/insertion/deletion anomalies.

</div>

<div class="fragment">

**2.** A **DBMS** is the software layer between users and data, providing storage, retrieval, transactions, concurrency, recovery, security, and integrity.

</div>

<div class="fragment">

**3.** The **ANSI-SPARC** three-schema architecture separates **external** (views), **conceptual** (enterprise), and **internal** (physical) levels — enabling data independence.

</div>

<div class="fragment">

**4.** **Logical data independence** is harder to achieve than physical because conceptual changes can ripple through queries.

</div>

<div class="fragment">

**5.** The **DBLC** is an iterative eight-phase process; requirements analysis is the most consequential phase.

</div>

<div class="fragment">

**6.** A **multi-user DBMS** adds concurrency control, transactions, recovery, and security — each with its own trade-offs.

</div>

---

## Next — Chapter 2

**The Relational Model.**

<div class="fragment">

- Data models (conceptual, logical, physical).
- **Relations**, **tuples**, **attributes**, **domains**.
- **Keys**: superkey, candidate, primary, foreign.
- **Entity-integrity** and **referential-integrity** constraints.
- **Schema** vs. **instance**.

</div>

<div class="fragment">

Chapter 1 described what a DBMS *is*. Chapter 2 describes the **data model** that almost every DBMS implements.

</div>