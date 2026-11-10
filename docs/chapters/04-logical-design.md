# Chapter 4 — Logical Database Design and Normalisation

!!! abstract "Chapter at a glance"
    **Coverage:** ER-to-relational mapping (7 steps) · EER-to-relational mapping (2 extra steps) · informal design guidelines · functional dependencies · normal forms 1NF, 2NF, 3NF, BCNF.

---

## Learning outcomes

1. Apply the 7-step ER-to-relational mapping algorithm.
2. Apply the 2 extra steps for EER constructs (specialisation, categories).
3. State and use Armstrong's axioms to compute closures and minimal covers.
4. Normalise a schema to 3NF and BCNF; verify decomposition quality.

---

## 4.1 The 7-step ER-to-relational mapping (Elmasri §9.1)

**Step 1 — Regular entity types.** Each regular entity becomes a relation. Include all simple attributes. Choose one key as primary key.

**Step 2 — Weak entity types.** Each weak entity becomes a relation. Include the owner's primary key as a foreign key. The primary key is (owner key, partial key).

**Step 3 — Binary 1:1 relationships.** Include the primary key of one side as a foreign key in the other side. Prefer the side with total participation.

**Step 4 — Binary 1:N relationships.** Include the primary key of the "1" side as a foreign key in the "N" side.

**Step 5 — Binary M:N relationships.** Create a new relation with foreign keys to both sides. Primary key is the combination of both foreign keys. Add any relationship attributes.

**Step 6 — Multivalued attributes.** Create a new relation with the attribute and the owner's primary key. Primary key is (owner key, attribute).

**Step 7 — N-ary relationships.** Create a new relation with foreign keys to all participants.

### 4.1.1 EER extra steps (Elmasri §9.2)

**Step 8 — Specialisation / generalisation.** Four options:

| Option | Description | When to use |
|---|---|---|
| 8A | Multiple relations (superclass + subclasses) | Any specialisation |
| 8B | Multiple relations (subclasses only) | Total, disjoint |
| 8C | Single relation with a type attribute | Disjoint, no subclass-specific attributes |
| 8D | Single relation with multiple type attributes | Overlapping |

**Step 9 — Categories.** Create one relation per subclass, and a new relation for the category that includes the primary key of each participant, plus a type discriminator.

---

## Worked example 4.1 — mapping the COMPANY database

**Input ER.** (From Chapter 3.)

**Step 1 — regular entities.**

```sql
CREATE TABLE EMPLOYEE (
    ssn        CHAR(9) PRIMARY KEY,
    name       VARCHAR(50),
    address    VARCHAR(100),
    salary     DECIMAL(10,2),
    sex        CHAR(1),
    birth_date DATE
);

CREATE TABLE DEPARTMENT (
    dnumber    INT PRIMARY KEY,
    dname      VARCHAR(50) UNIQUE
);

CREATE TABLE PROJECT (
    pnumber    INT PRIMARY KEY,
    pname      VARCHAR(50),
    location   VARCHAR(50)
);
```

**Step 3 — 1:1 `MANAGES`.** Put `mgr_ssn` (foreign key to EMPLOYEE) in DEPARTMENT.

```sql
ALTER TABLE DEPARTMENT ADD COLUMN mgr_ssn CHAR(9),
    ADD FOREIGN KEY (mgr_ssn) REFERENCES EMPLOYEE(ssn);
```

**Step 4 — 1:N `WORKS_FOR` and `CONTROLS`.** Add foreign keys on the "N" side.

```sql
ALTER TABLE EMPLOYEE ADD COLUMN dno INT,
    ADD FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber);

ALTER TABLE PROJECT ADD COLUMN dnum INT,
    ADD FOREIGN KEY (dnum) REFERENCES DEPARTMENT(dnumber);
```

**Step 5 — M:N `WORKS_ON`.**

```sql
CREATE TABLE WORKS_ON (
    essn CHAR(9),
    pno  INT,
    hours DECIMAL(3,1),
    PRIMARY KEY (essn, pno),
    FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn),
    FOREIGN KEY (pno)  REFERENCES PROJECT(pnumber)
);
```

**Step 6 — multivalued attribute `locations` of DEPARTMENT.**

```sql
CREATE TABLE DEPT_LOCATIONS (
    dnumber   INT,
    dlocation VARCHAR(50),
    PRIMARY KEY (dnumber, dlocation),
    FOREIGN KEY (dnumber) REFERENCES DEPARTMENT(dnumber)
);
```

**Step 2 — weak entity `DEPENDENT`.**

```sql
CREATE TABLE DEPENDENT (
    essn           CHAR(9),
    dependent_name VARCHAR(50),
    sex            CHAR(1),
    birth_date     DATE,
    relationship   VARCHAR(20),
    PRIMARY KEY (essn, dependent_name),
    FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn)
);
```

**Result: seven relations** — EMPLOYEE, DEPARTMENT, PROJECT, WORKS_ON, DEPT_LOCATIONS, DEPENDENT (and the EER subclasses to be added in Step 8).

---

## 4.2 Informal design guidelines (Elmasri §14.1)

Three red flags in a relational design:

1. **Attribute semantics** — each attribute should describe a single, clearly-defined property of the entity. Mixing semantics (e.g., a `phone` attribute that sometimes holds a home number, sometimes a work number) confuses joins.
2. **Redundant information** — duplicating data in tuples causes update anomalies (insertion, deletion, modification).
3. **NULL values** — excessive NULLs waste space and complicate joins; they also make constraints and aggregates hard to interpret.

**Spurious tuples** — decompositions that produce extra rows on join indicate a bad design. A **lossless-join decomposition** is one that avoids spurious tuples.

---

## 4.3 Functional dependencies (Elmasri §14.2)

!!! abstract "Definition"
    A **functional dependency** (FD) $X \to Y$ on relation $R$ holds if, for every two tuples $t_1, t_2$ with $t_1[X] = t_2[X]$, we also have $t_1[Y] = t_2[Y]$.

**Armstrong's axioms** (sound and complete):

- **Reflexivity:** if $Y \subseteq X$, then $X \to Y$.
- **Augmentation:** if $X \to Y$, then $XZ \to YZ$.
- **Transitivity:** if $X \to Y$ and $Y \to Z$, then $X \to Z$.

**Derived rules:**

- **Union:** $X \to Y, X \to Z \Rightarrow X \to YZ$.
- **Decomposition:** $X \to YZ \Rightarrow X \to Y, X \to Z$.
- **Pseudotransitivity:** $X \to Y, WY \to Z \Rightarrow WX \to Z$.

**Closure.** The **closure $X^+$** of $X$ under $F$ is the set of all attributes functionally determined by $X$.

**Minimal cover.** A set $F_{\min}$ equivalent to $F$ with:

- No redundant attributes (every dependency is needed).
- No redundant attributes on the left side.
- RHS has a single attribute per dependency.

---

## Worked example 4.2 — closure and minimal cover

**Given.** $R(A,B,C,D,E)$ with $F = \{AB \to C,\ C \to D,\ D \to E\}$.

**Compute $(AB)^+$:**

- $AB$ (initial)
- $+ C$ (from $AB \to C$)
- $+ D$ (from $C \to D$)
- $+ E$ (from $D \to E$)

So $(AB)^+ = \{A, B, C, D, E\}$ — $AB$ is a superkey.

**Is $AB$ a candidate key?** Check $(A)^+ = \{A\}$ and $(B)^+ = \{B\}$ — neither is a superkey. **$AB$ is a candidate key.** It is the only candidate key (nothing else determines $A$ or $B$).

**Minimal cover of $F$:** $F$ is already minimal — no dependency can be removed without losing equivalence, and the left sides are single-attribute or minimal.

---

## 4.4 Normal forms (Elmasri §14.3–§14.5)

### 4.4.1 1NF — first normal form

!!! note "1NF"
    Every attribute value is **atomic** — no sets, lists, or nested relations.

**Fix:** move multivalued attributes into a separate relation.

### 4.4.2 2NF — second normal form

!!! note "2NF"
    In 1NF, **and** every non-prime attribute is **fully functionally dependent** on every candidate key (no partial dependency on part of a key).

**Fix:** decompose so partial dependencies go to their own relations.

### 4.4.3 3NF — third normal form

!!! note "3NF"
    In 2NF, **and** no non-prime attribute is **transitively dependent** on any candidate key. Equivalently, for every non-trivial FD $X \to Y$: **$X$ is a superkey, or $Y$ is a prime attribute**.

### 4.4.4 BCNF — Boyce-Codd normal form

!!! note "BCNF"
    For every non-trivial FD $X \to Y$: **$X$ is a superkey**.

BCNF is stricter than 3NF: it removes all FDs based on non-superkey determinants, even if the RHS is prime.

---

## Worked example 4.3 — 3NF synthesis

**Given.** $R(A,B,C,D,E)$ with $F = \{AB \to C,\ C \to D,\ D \to E\}$.

**Step 1 — minimal cover.** $F$ is minimal.

**Step 2 — combine FDs with the same LHS.** None shared.

**Step 3 — create a relation per dependency:**

- $R_1(A, B, C)$
- $R_2(C, D)$
- $R_3(D, E)$

**Step 4 — add a relation for any candidate key not covered.** $AB$ is covered by $R_1$.

**Result.** $R_1, R_2, R_3$ — in 3NF (each FD has a superkey or prime RHS) and **dependency-preserving**.

**Is it BCNF?** No. $C \to D$ has $C$ not a superkey of $R_2$. Decompose $R_2$ into $R_2'(C)$ — but that loses the dependency. So 3NF is the best we can do while preserving dependencies.

---

## 4.5 Practical use of normal forms

- **3NF** is the practical target for most OLTP systems — it removes redundancy while preserving dependencies.
- **BCNF** removes additional redundancy but may lose dependency preservation (may require join to check a dependency).
- **Denormalisation** is sometimes used in data warehouses — deliberately introduce redundancy for query performance.

---

## 4.6 Quiz

1. List the 7 steps of the ER-to-relational mapping.
2. Map an ER model with a weak entity to relations.
3. State Armstrong's three axioms.
4. Compute the closure of $AB$ given $AB \to C, C \to D, D \to E$.
5. Distinguish 3NF from BCNF. Give an example of a relation that is in 3NF but not in BCNF.

---

## Answer key

??? success "Click to reveal"

    1. Regular entities → weak entities → 1:1 → 1:N → M:N → multivalued → n-ary.
    2. Weak entity gets a relation with (owner PK, partial key) as PK.
    3. Reflexivity, augmentation, transitivity.
    4. $AB^+ = \{A,B,C,D,E\}$.
    5. 3NF allows $Y$ prime in $X \to Y$; BCNF requires $X$ superkey. Example: $R(A,B,C)$ with $AB \to C, C \to B$. Candidate keys: $AB, AC$. $C \to B$ has $C$ not a superkey, $B$ prime → 3NF but not BCNF.

---

## Further reading

- Elmasri & Navathe, §9.1–§9.2, §14.1–§14.6.
- Silberschatz et al., §7.1–§7.7.