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
  js: assets/lecture-02.js

---

# ECEG-4191

## Chapter 2 — The Relational Model

---

## Learning outcomes

<div class="fragment">

1. Classify the three types of **data models** (conceptual, logical, physical).

</div>

<div class="fragment">

2. Define a **relation** formally and identify its components: tuples, attributes, domains, degree, cardinality.

</div>

<div class="fragment">

3. Distinguish **superkey**, **candidate key**, **primary key**, and **foreign key**.

</div>

<div class="fragment">

4. State and apply the **entity-integrity** and **referential-integrity** constraints.

</div>

<div class="fragment">

5. Distinguish a **schema** from an **instance**.

</div>

---

## 2.1 — Data Models

???+ note "Definition"
    A **data model** is a collection of concepts for describing data, relationships, semantics, and constraints.

<div class="fragment">

???+ info "Three categories"
    | Category | Purpose | Examples |
    |---|---|---|
    | **Conceptual** | High-level, user-oriented | ER model, EER model (Chapter 3) |
    | **Logical** | DBMS-implementable | Relational, object-oriented, object-relational |
    | **Physical** | Storage-level | Files, indexes, page layouts (Chapter 7) |

</div>

<div class="fragment">

!!! tip "Each model has its own purpose"
    - Conceptual models are for **communication** with users.
    - Logical models are for **implementation**.
    - Physical models are for **performance**.

</div>

---

## 2.2 — The Relational Model

???+ note "Definition (Codd 1970; Elmasri §3.1)"
    A **relation** $r$ over schema $R(A_1, A_2, \ldots, A_n)$ is a subset of the Cartesian product of domains $D_1 \times D_2 \times \cdots \times D_n$.

<div class="fragment">

???+ info "What this means in practice"
    - A relation is a **set of tuples** — each tuple belongs to exactly one combination of domain values.
    - The **order** of tuples is irrelevant.
    - Each attribute value is **atomic** — no nested relations, no sets.

</div>

<div class="fragment">

!!! warning "Codd's requirements"
    - **No duplicate tuples** — a relation is a set, not a multiset.
    - **No ordering of tuples** — mathematically, there is no "first" tuple.
    - **No ordering of attributes** — although practical DBMSes fix a presentation order.

</div>

---

## 2.3 — Terminology

???+ note "The formal/informal mapping"
    | Formal | Informal | Example |
    |---|---|---|
    | Relation | Table | `EMPLOYEE` |
    | Tuple | Row | `(101, 'Alice', 50000, 10)` |
    | Attribute | Column | `name` |
    | Domain | Data type + constraints | `INTEGER`, `VARCHAR(50)` |
    | Degree | Number of columns | 4 |
    | Cardinality | Number of rows | 1,000 |

<div class="fragment">

???+ info "The informal terms are fine for conversation"
    "Table", "row", and "column" are used informally everywhere — including in SQL documentation. "Relation", "tuple", and "attribute" are used in the formal theory and in query optimization.

</div>

---

## 2.4 — A Relation Instance

???+ note "Example: `EMPLOYEE` relation"
    | emp_id | ssn | name | salary | dept_id |
    |:---:|---|---|---:|:---:|
    | 101 | 111-22-3333 | Alice | 50,000 | 10 |
    | 102 | 222-33-4444 | Bob | 60,000 | 20 |
    | 103 | 333-44-5555 | Carol | 55,000 | 10 |
    | 104 | 444-55-6666 | David | 45,000 | 20 |
    | 105 | 555-66-7777 | Eve | 70,000 | 30 |

<div class="fragment">

???+ info "Vocabulary check"
    - **Degree** = 5 (five attributes).
    - **Cardinality** = 5 (five tuples).
    - **Domain of `salary`** = non-negative integers (or a DECIMAL type with a range).
    - **Primary key** = `emp_id`.

</div>

<aside class="notes">
  <div style="font-size: 16px; line-height: 1.6;">
Walk through this table slowly. Ask: what would happen if we allowed duplicate tuples? Then the "relation as set" property breaks, and operations like union, difference, and division become ambiguous. This is why SQL's default (bag semantics) is a departure from Codd's pure model, requiring DISTINCT.
  </div>
</aside>

---

## 2.5 — Keys: The Four Levels

???+ abstract "Superkey"
    Any set of attributes $K \subseteq R$ such that no two tuples agree on $K$.

<div class="fragment">

???+ abstract "Candidate key"
    A **minimal** superkey — no proper subset of $K$ is a superkey.

</div>

<div class="fragment">

???+ abstract "Primary key"
    A **chosen** candidate key (the "official" identifier).

</div>

<div class="fragment">

???+ abstract "Foreign key"
    An attribute (or set) in relation $R_1$ that references the **primary key** of another relation $R_2$.

</div>

<div class="fragment">

!!! tip "The hierarchy"
    Every **candidate key** is a **superkey**, but not vice versa. The primary key is one of the candidate keys. Foreign keys reference primary keys.

</div>

---

## 2.6 — Keys: Worked Example

???+ note "Given relation `EMPLOYEE(emp_id, ssn, name, dept_id)`"
    | Key type | Example | Why |
    |---|---|---|
    | **Superkey** | `(emp_id)`, `(ssn)`, `(emp_id, ssn)`, `(emp_id, name)`, `(ssn, name)` | All uniquely identify a tuple |
    | **Candidate key** | `emp_id`, `ssn` | Minimal — removing any attribute loses uniqueness |
    | **Primary key** | `emp_id` | Chosen as the official identifier |
    | **Foreign key** | `dept_id` | References `DEPARTMENT.dept_id` |

<div class="fragment">

???+ warning "Not every superkey is a candidate key"
    `(emp_id, ssn)` is a superkey, but **not** a candidate key — because `emp_id` alone is already a superkey. The minimal ones are `emp_id` and `ssn`.

</div>

---

## 2.7 — Entity-Integrity Constraint

???+ abstract "Entity-integrity constraint (Elmasri §3.2)"
    **No primary-key attribute may be NULL.** Every tuple must have a unique, non-null identifier.

<div class="fragment">

???+ info "Why this is necessary"
    If a primary key could be NULL:
    - Multiple tuples could have NULL primary keys (NULL = NULL is unknown, not true).
    - The tuple would have no identifier.
    - Foreign keys referencing it would be ambiguous.

</div>

<div class="fragment">

???+ warning "NULL is not a value"
    In SQL, NULL means "unknown" or "does not exist". NULL = NULL evaluates to **UNKNOWN**, not TRUE. This is why you use `IS NULL`, not `= NULL`.

</div>

---

## 2.8 — Referential-Integrity Constraint

???+ abstract "Referential-integrity constraint (Elmasri §3.2)"
    For a foreign key $FK$ in $R_1$ referencing primary key $PK$ in $R_2$:
    
    For every tuple $t_1 \in R_1$, either:
    - $t_1[FK]$ is NULL, **or**
    - there exists a tuple $t_2 \in R_2$ with $t_2[PK] = t_1[FK]$.

<div class="fragment">

???+ note "What it prevents"
    **Dangling references.** Without the constraint, `EMPLOYEE.dept_id = 99` could exist even though no `DEPARTMENT` with `dept_id = 99` exists.

</div>

<div class="fragment">

!!! tip "The NULL allowance"
    A foreign key **may be NULL** — this represents "no reference" (e.g., a new employee not yet assigned to a department). But it must not point to a non-existent primary key.

</div>

---

## 2.9 — Referential Actions

???+ note "What happens on DELETE / UPDATE of the referenced key?"
    | Action | Effect on the foreign key |
    |---|---|
    | **RESTRICT / NO ACTION** | Reject the operation |
    | **CASCADE** | Delete / update the referencing tuple |
    | **SET NULL** | Set the foreign key to NULL |
    | **SET DEFAULT** | Set the foreign key to its default |

<div class="fragment">

???+ info "Example: `DEPARTMENT` is deleted"
    | Action | Result |
    |---|---|
    | RESTRICT | Delete fails — departments with employees cannot be deleted |
    | CASCADE | Every employee in the department is also deleted |
    | SET NULL | Employees' `dept_id` becomes NULL |
    | SET DEFAULT | Employees' `dept_id` moves to a default department |

</div>

<div class="fragment">

!!! warning "CASCADE is dangerous"
    A single `DELETE FROM DEPARTMENT WHERE dept_id = 10` could delete hundreds of employees. Use CASCADE only when the relationship is **compositional** (the dependent cannot exist independently).

</div>

---

## 2.10 — Schema vs Instance

???+ note "Schema vs instance"
    | Term | Definition | Changes |
    |---|---|---|
    | **Schema** | The structure — relation names, attribute names, types, constraints | Rarely |
    | **Instance** | The current set of tuples | Constantly |

<div class="fragment">

???+ info "Example schema"
    ```sql
    CREATE TABLE EMPLOYEE (
        emp_id  INTEGER PRIMARY KEY,
        name    VARCHAR(50) NOT NULL,
        salary  DECIMAL(10,2) CHECK (salary >= 0),
        dept_id INTEGER,
        FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
    );
    ```
    
    This is the **schema** — defined once, changed rarely.

</div>

<div class="fragment">

???+ info "Example instance"
    | emp_id | name | salary | dept_id |
    |:---:|---|---:|---:|
    | 101 | Alice | 50,000 | 10 |
    | 102 | Bob | 60,000 | 20 |
    
    The **instance** changes with every insert, update, delete.

</div>

---

## 2.11 — Schema Notation

???+ note "Formal notation"
    A relation schema is written:
    
    $$
    R(A_1, A_2, \ldots, A_n)
    $$
    
    with **underlined primary key**:
    
    $$
    \text{EMPLOYEE}(\underline{\text{emp\_id}}, \text{ssn}, \text{name}, \text{salary}, \text{dept\_id})
    $$

<div class="fragment">

???+ info "Foreign keys are shown with arrows"
    $$
    \text{EMPLOYEE}(\underline{\text{emp\_id}}, \text{name}, \text{salary}, \text{dept\_id} \to \text{DEPARTMENT})
    $$

</div>

<div class="fragment">

!!! tip "This notation is the basis for everything in Chapter 4"
    ER-to-relational mapping (Chapter 4) produces exactly this notation. Normalization (Chapter 4) operates on these schemas.

</div>

---

## 2.12 — Relational Algebra Preview

???+ note "What operations does the model support?"
    | Operation | Symbol | Purpose |
    |---|---|---|
    | Selection | $\sigma$ | Filter tuples |
    | Projection | $\pi$ | Select columns |
    | Union | $\cup$ | Combine two relations |
    | Difference | $-$ | Tuples in one but not other |
    | Cartesian product | $\times$ | Every pairing |
    | Join | $\bowtie$ | Combine on matching values |

<div class="fragment">

!!! tip "Chapter 5 develops these in full"
    For now, understand that the relational model is not just data structures — it comes with a **query algebra**. That is what makes it uniquely powerful.

</div>

---

## Worked example 2.1 — Identify all key types

!!! tip "Problem:"
    Given:
    
    ```
    STUDENT(student_id, ssn, name, dept, advisor_id)
    COURSE(course_id, title, credits)
    ENROLL(student_id, course_id, grade, semester)
    ```
    
    Identify superkeys, candidate keys, primary keys, and foreign keys.

<div class="fragment">

???+ note "**STUDENT**"
    - **Candidate keys:** `student_id`, `ssn`.
    - **Primary key:** `student_id`.
    - **Foreign key:** `advisor_id` → `STUDENT.student_id`.

</div>

<div class="fragment">

??? info "**COURSE**"
    - **Candidate keys:** `course_id`.
    - **Primary key:** `course_id`.

</div>

<div class="fragment">

??? info "**ENROLL**"
    - **Candidate key:** `(student_id, course_id, semester)` — because a student can take the same course in different semesters.
    - **Primary key:** `(student_id, course_id, semester)`.
    - **Foreign keys:** `student_id` → `STUDENT`, `course_id` → `COURSE`.

</div>

<div class="fragment">

??? success "**Conclusion.**"
    Every table has exactly one primary key; foreign keys establish references; **composite keys** (ENROLL) arise when a single attribute is insufficient.

</div>

---

## Worked example 2.2 — Detect a constraint violation

!!! tip "Problem:"
    Given:
    
    ```
    EMPLOYEE(emp_id, name, dept_id)   -- dept_id → DEPARTMENT
    DEPARTMENT(dept_id, dname)
    ```
    
    Current state:
    
    | EMPLOYEE | emp_id | name | dept_id |
    |---|---|---|---|
    | 1 | Alice | 10 |
    | 2 | Bob | 20 |
    | 3 | Carol | 99 |
    
    | DEPARTMENT | dept_id | dname |
    |---|---|---|
    | 10 | Sales |
    | 20 | Engineering |
    
    Which constraint is violated?

<div class="fragment">

???+ note "**Step 1 — check entity integrity.**"
    All `emp_id` values are non-NULL and unique → **satisfied**.

</div>

<div class="fragment">

??? info "**Step 2 — check referential integrity.**"
    - `Alice → 10`: 10 exists in DEPARTMENT ✓.
    - `Bob → 20`: 20 exists in DEPARTMENT ✓.
    - `Carol → 99`: 99 does NOT exist in DEPARTMENT → **violation**.

</div>

<div class="fragment">

??? success "**Conclusion.**"
    `Carol`'s `dept_id = 99` is a dangling reference. The DBMS would reject this tuple (unless `dept_id` were NULL).

</div>

---

## 2.13 — Why the Relational Model Dominates

???+ info "Six reasons"
    | Reason | Explanation |
    |---|---|
    | **Simple** | One data structure: the relation |
    | **Mathematically grounded** | Set theory + first-order logic |
    | **Query algebra** | Provably equivalent to relational calculus |
    | **Data independence** | Physical storage is hidden |
    | **Standardized** | SQL (Chapter 6) |
    | **Implementable** | Efficient storage and indexing (Chapter 7) |

<div class="fragment">

!!! tip "The historical argument"
    Codd's 1970 paper was a **manifesto**: it argued that a simple data model with a formal algebra could replace the tangled hierarchical and network models. Fifty years later, it has won.

</div>

---

## Chapter 2 — key takeaways

<div class="fragment">

**1.** Three categories of data model: **conceptual** (ER), **logical** (relational), **physical** (files).

</div>

<div class="fragment">

**2.** A **relation** is a set of tuples over a schema; each attribute takes values from a **domain**.

</div>

<div class="fragment">

**3.** Four key levels: **superkey**, **candidate key**, **primary key**, **foreign key**. Every candidate is a superkey, but not vice versa.

</div>

<div class="fragment">

**4.** **Entity-integrity:** primary keys are never NULL. **Referential-integrity:** foreign keys either are NULL or match a primary key.

</div>

<div class="fragment">

**5.** **Referential actions** (RESTRICT, CASCADE, SET NULL, SET DEFAULT) determine what happens on delete/update of the referenced key.

</div>

<div class="fragment">

**6.** **Schema** vs **instance** — structure vs. current data. The schema changes rarely; the instance constantly.

</div>

---

## Next — Chapter 3

**Conceptual Design and E-R Modelling.**

<div class="fragment">

- The three design phases.
- **Entities**, **attributes**, **relationships**.
- **Cardinality ratios** and **participation constraints**.
- **Weak entities** and identifying relationships.
- **EER concepts:** subclasses, specialisation, generalisation, categories.
- The **COMPANY** case study.

</div>

<div class="fragment">

Chapter 2 described the target model. Chapter 3 describes how to **design** a database from requirements — before writing a single line of SQL.

</div>