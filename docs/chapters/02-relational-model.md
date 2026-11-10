# Chapter 2 — The Relational Model

!!! abstract "Chapter at a glance"
    **Coverage:** data models · relational model · relational database · terminology · relational constraints · schema and instances.

---

## Learning outcomes

1. Define the relational model in terms of relation, tuple, attribute, and domain.
2. Identify superkeys, candidate keys, primary keys, and foreign keys.
3. State and apply the entity-integrity and referential-integrity constraints.
4. Distinguish schema from instance.

---

## 2.1 Data models

A **data model** is a set of concepts for describing data, relationships, semantics, and constraints.

| Category | Examples |
|---|---|
| **Conceptual** | ER model, EER model (Chapter 3) |
| **Logical** | Relational, object-oriented, object-relational |
| **Physical** | Files, indexes (Chapter 7) |

The **relational model** is the dominant logical model — nearly every commercial DBMS implements it.

---

## 2.2 The relational model

!!! note "Definition (Elmasri §3.1, Codd 1970)"
    A **relation** is a set of tuples over a schema $R(A_1, A_2, \ldots, A_n)$, where each attribute $A_i$ takes values from a **domain** $D_i$.

Formally, a relation $r$ on schema $R$ is a subset of the Cartesian product $D_1 \times D_2 \times \cdots \times D_n$.

**Key properties (Codd's requirements):**

- Each tuple is **unique** (a relation is a set of tuples).
- Tuples are **unordered**.
- Attributes are **unordered** (although column order may be presented).
- Each attribute value is **atomic** (1NF — see Chapter 4).

---

## 2.3 Terminology

| Formal term | Informal equivalent |
|---|---|
| Relation | Table |
| Tuple | Row |
| Attribute | Column |
| Domain | Data type (permitted values) |
| Degree | Number of attributes |
| Cardinality | Number of tuples |

---

## 2.4 Keys

| Term | Definition |
|---|---|
| **Superkey** | Any set of attributes that uniquely identifies a tuple |
| **Candidate key** | A **minimal** superkey (no attribute can be removed) |
| **Primary key** | A chosen candidate key |
| **Foreign key** | Attribute(s) in one relation that match a primary key in another |

**Example.** In `EMPLOYEE(emp_id, ssn, name, dept_id)`:

- Superkeys: `emp_id`, `ssn`, `(emp_id, ssn)`, `(emp_id, name)`, …
- Candidate keys: `emp_id`, `ssn`.
- Primary key: `emp_id` (chosen).
- Foreign key: `dept_id` references `DEPARTMENT`.

---

## 2.5 Relational constraints

Two key constraints (Elmasri §3.2):

!!! abstract "Entity-integrity constraint"
    No primary-key attribute may be NULL. Every tuple must have a unique, non-null identifier.

!!! abstract "Referential-integrity constraint"
    For a foreign key $FK$ in relation $R_1$ referencing primary key $PK$ in $R_2$: for every tuple in $R_1$, either $FK$ is NULL, or $FK$ matches a $PK$ value in $R_2$.

**Referential-action options:**

| Action | On DELETE / UPDATE of referenced key |
|---|---|
| `RESTRICT` / `NO ACTION` | Reject |
| `CASCADE` | Propagate the change |
| `SET NULL` | Set the foreign key to NULL |
| `SET DEFAULT` | Set the foreign key to its default |

---

## 2.6 Schema vs. instance

| Term | Definition |
|---|---|
| **Schema** | The structure — relation names, attribute names, types, constraints (rarely changes) |
| **Instance** | The current set of tuples (changes constantly) |

**Example schema:**

```sql
CREATE TABLE EMPLOYEE (
    emp_id   INTEGER PRIMARY KEY,
    name     VARCHAR(50) NOT NULL,
    dept_id  INTEGER,
    FOREIGN KEY (dept_id) REFERENCES DEPARTMENT(dept_id)
);
```

**Example instance:**

| emp_id | name | dept_id |
|---|---|---|
| 101 | Alice | 10 |
| 102 | Bob | 20 |
| 103 | Carol | 10 |

---

## 2.7 Quiz

1. Define relation, tuple, attribute, and domain.
2. What is the difference between a superkey, a candidate key, and a primary key?
3. State the entity-integrity and referential-integrity constraints.
4. Give one example of a schema and one of an instance.
5. Distinguish the four referential actions (RESTRICT, CASCADE, SET NULL, SET DEFAULT).

---

## Answer key

??? success "Click to reveal"

    1. Relation = table; tuple = row; attribute = column; domain = set of permitted values for an attribute.
    2. Superkey uniquely identifies a tuple; candidate key is a minimal superkey; primary key is a chosen candidate key.
    3. Entity: no primary-key attribute is NULL. Referential: foreign key is NULL or matches a primary-key value in the referenced relation.
    4. (See §2.6.)
    5. RESTRICT: reject. CASCADE: propagate. SET NULL: nullify. SET DEFAULT: replace with default value.

---

## Further reading

- Elmasri & Navathe, §3.1–§3.5.
- Silberschatz et al., §2.1–§2.5.