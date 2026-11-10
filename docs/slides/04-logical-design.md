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
  js: assets/lecture-04.js

---

# ECEG-4191

## Chapter 4 — Logical Design and Normalisation

---

## Learning outcomes

<div class="fragment">

1. Apply the **7-step ER-to-relational mapping** to any ER diagram.

</div>

<div class="fragment">

2. Apply the **2 extra steps** for EER constructs (specialisation, categories).

</div>

<div class="fragment">

3. Compute **closures** and **minimal covers** for sets of functional dependencies.

</div>

<div class="fragment">

4. Determine whether a schema is in **1NF, 2NF, 3NF, BCNF**.

</div>

<div class="fragment">

5. Decompose a schema into 3NF or BCNF while preserving properties.

</div>

---

## 4.1 — ER-to-Relational Mapping Overview

???+ note "The 7 steps (Elmasri §9.1)"
    | Step | Maps |
    |---|---|
    | **1** | Regular entity types |
    | **2** | Weak entity types |
    | **3** | Binary 1:1 relationships |
    | **4** | Binary 1:N relationships |
    | **5** | Binary M:N relationships |
    | **6** | Multivalued attributes |
    | **7** | N-ary relationships |

<div class="fragment">

!!! tip "The order matters"
    Steps 1–2 handle entities and their attributes. Steps 3–5 handle relationships. Steps 6–7 handle the "extras". Following the order avoids duplicate table creation.

</div>

---

## 4.2 — Step 1: Regular Entity Types

???+ note "Rule"
    Each regular entity becomes a **relation**:
    - Include all **simple** attributes.
    - For each **composite** attribute, include only its simple components.
    - Choose one **key** attribute as the primary key.

<div class="fragment">

???+ info "Example: `EMPLOYEE(ssn, name(first, middle, last), sex, address, salary, bdate)`"
    Becomes:
    
    ```sql
    CREATE TABLE EMPLOYEE (
        ssn     CHAR(9) PRIMARY KEY,
        first_name VARCHAR(30),
        middle_name VARCHAR(30),
        last_name   VARCHAR(30),
        sex     CHAR(1),
        address VARCHAR(100),
        salary  DECIMAL(10,2),
        bdate   DATE
    );
    ```

</div>

<div class="fragment">

!!! warning "Don't create a table for the composite attribute"
    `name` is composite, but it is **not** a separate table. Its **simple components** (`first_name`, `middle_name`, `last_name`) become columns of `EMPLOYEE`.

</div>

---

## 4.3 — Step 2: Weak Entity Types

???+ note "Rule"
    Each weak entity becomes a relation with:
    - Its own attributes.
    - The **owner's primary key** as a foreign key.
    - **Primary key** = (owner's primary key, partial key).

<div class="fragment">

???+ info "Example: `DEPENDENT`"
    ```sql
    CREATE TABLE DEPENDENT (
        essn            CHAR(9),
        dependent_name  VARCHAR(50),
        sex             CHAR(1),
        birth_date      DATE,
        relationship    VARCHAR(20),
        PRIMARY KEY (essn, dependent_name),
        FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn)
    );
    ```

</div>

<div class="fragment">

!!! tip "The full key is composite"
    `dependent_name` is a **partial key** — unique only within one employee. The full key must include `essn` to be globally unique.

</div>

---

## 4.4 — Step 3: Binary 1:1 Relationships

???+ note "Rule"
    Put the primary key of one side as a **foreign key** in the other side.
    
    **Prefer the side with total participation** — that ensures every tuple on that side has a non-NULL reference.

<div class="fragment">

???+ info "Example: `EMPLOYEE MANAGES DEPARTMENT` (1:1)"
    Both sides are partial in the COMPANY model. Standard choice: put the foreign key in `DEPARTMENT`:
    
    ```sql
    ALTER TABLE DEPARTMENT
        ADD COLUMN mgr_ssn CHAR(9),
        ADD FOREIGN KEY (mgr_ssn) REFERENCES EMPLOYEE(ssn);
    ```

</div>

<div class="fragment">

!!! warning "Why NOT in EMPLOYEE?"
    If `mgr_dno` were in `EMPLOYEE`, most employees would have NULL there (only one manager per department). Putting the reference in `DEPARTMENT` is more natural.

</div>

<div class="fragment">

???+ note "The cross-reference option"
    An alternative is a **cross-reference table** with both foreign keys. This is used when neither side has total participation and the relationship has its own attributes.

</div>

---

## 4.5 — Step 4: Binary 1:N Relationships

???+ note "Rule"
    Put the **primary key of the "1" side** as a **foreign key** in the **"N" side**.
    
    This is the most common mapping — every `EMPLOYEE` has one `dno`.

<div class="fragment">

???+ info "Example: `EMPLOYEE WORKS_FOR DEPARTMENT` (N:1)"
    ```sql
    ALTER TABLE EMPLOYEE
        ADD COLUMN dno INT,
        ADD FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber);
    ```

</div>

<div class="fragment">

!!! tip "Total participation ⇒ NOT NULL"
    Because every employee must work for a department (total participation), add `NOT NULL`:
    
    ```sql
    ALTER TABLE EMPLOYEE ADD COLUMN dno INT NOT NULL;
    ```

</div>

---

## 4.6 — Step 5: Binary M:N Relationships

???+ note "Rule"
    Create a **new relation** with:
    - The primary keys of **both sides** as foreign keys.
    - Any **relationship attributes** (e.g., `hours`).
    - **Primary key** = the combination of both foreign keys.

<div class="fragment">

???+ info "Example: `EMPLOYEE WORKS_ON PROJECT` (M:N with `hours`)"
    ```sql
    CREATE TABLE WORKS_ON (
        essn  CHAR(9),
        pno   INT,
        hours DECIMAL(3,1),
        PRIMARY KEY (essn, pno),
        FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn),
        FOREIGN KEY (pno)  REFERENCES PROJECT(pnumber)
    );
    ```

</div>

<div class="fragment">

!!! warning "M:N cannot be a foreign key on either side"
    A single `EMPLOYEE.ssn` cannot reference many projects, and a single `PROJECT.pnumber` cannot reference many employees. The cross-reference table is **required**.

</div>

---

## 4.7 — Step 6: Multivalued Attributes

???+ note "Rule"
    Create a **new relation** with:
    - The **multivalued attribute** itself.
    - The **owner's primary key** as a foreign key.
    - **Primary key** = (owner's primary key, attribute).

<div class="fragment">

???+ info "Example: `DEPARTMENT.locations` (multivalued)"
    ```sql
    CREATE TABLE DEPT_LOCATIONS (
        dnumber   INT,
        dlocation VARCHAR(50),
        PRIMARY KEY (dnumber, dlocation),
        FOREIGN KEY (dnumber) REFERENCES DEPARTMENT(dnumber)
    );
    ```

</div>

<div class="fragment">

!!! tip "Why not keep it in DEPARTMENT?"
    A relational attribute must be **atomic** (1NF). A multivalued attribute would violate 1NF. The separate table is the standard fix.

</div>

---

## 4.8 — Step 7: N-ary Relationships

???+ note "Rule"
    Create a **new relation** with:
    - The primary keys of **all participating entities** as foreign keys.
    - Any relationship attributes.
    - **Primary key** = the combination of all foreign keys (or a subset if some participants are functionally determined).

<div class="fragment">

???+ info "Example: `SUPPLIES(supplier, project, part, quantity)`"
    ```sql
    CREATE TABLE SUPPLIES (
        supplier_id INT,
        project_id  INT,
        part_id     INT,
        quantity    INT,
        PRIMARY KEY (supplier_id, project_id, part_id),
        FOREIGN KEY (supplier_id) REFERENCES SUPPLIER(supplier_id),
        FOREIGN KEY (project_id)  REFERENCES PROJECT(project_id),
        FOREIGN KEY (part_id)     REFERENCES PART(part_id)
    );
    ```

</div>

<div class="fragment">

!!! warning "Ternary ≠ three binary relationships"
    A ternary `SUPPLIES` means "supplier supplies part **to** project" — a joint constraint. Three separate binary relationships would lose the joint semantics.

</div>

---

## Worked example 4.1 — Map the COMPANY database

!!! tip "Problem:"
    Map the COMPANY ER model from Chapter 3 to a relational schema.

<div class="fragment">

???+ note "**Step 1 — regular entities.**"
    ```sql
    CREATE TABLE EMPLOYEE (
        ssn CHAR(9) PRIMARY KEY,
        first_name VARCHAR(30), middle_name VARCHAR(30), last_name VARCHAR(30),
        sex CHAR(1), address VARCHAR(100), salary DECIMAL(10,2), bdate DATE
    );
    CREATE TABLE DEPARTMENT (
        dnumber INT PRIMARY KEY,
        dname VARCHAR(50) UNIQUE
    );
    CREATE TABLE PROJECT (
        pnumber INT PRIMARY KEY,
        pname VARCHAR(50), location VARCHAR(50)
    );
    ```

</div>

<div class="fragment">

??? info "**Step 2 — weak entity.**"
    ```sql
    CREATE TABLE DEPENDENT (
        essn CHAR(9),
        dependent_name VARCHAR(50),
        sex CHAR(1), birth_date DATE, relationship VARCHAR(20),
        PRIMARY KEY (essn, dependent_name),
        FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn)
    );
    ```

</div>

<div class="fragment">

??? info "**Step 3 — 1:1 `MANAGES`.**"
    ```sql
    ALTER TABLE DEPARTMENT ADD COLUMN mgr_ssn CHAR(9),
        ADD FOREIGN KEY (mgr_ssn) REFERENCES EMPLOYEE(ssn);
    ```

</div>

<div class="fragment">

??? info "**Step 4 — 1:N `WORKS_FOR`, `CONTROLS`, `SUPERVISES`.**"
    ```sql
    ALTER TABLE EMPLOYEE ADD COLUMN dno INT,
        ADD FOREIGN KEY (dno) REFERENCES DEPARTMENT(dnumber);
    ALTER TABLE EMPLOYEE ADD COLUMN super_ssn CHAR(9),
        ADD FOREIGN KEY (super_ssn) REFERENCES EMPLOYEE(ssn);
    ALTER TABLE PROJECT ADD COLUMN dnum INT,
        ADD FOREIGN KEY (dnum) REFERENCES DEPARTMENT(dnumber);
    ```

</div>

<div class="fragment">

??? info "**Step 5 — M:N `WORKS_ON`.**"
    ```sql
    CREATE TABLE WORKS_ON (
        essn CHAR(9), pno INT, hours DECIMAL(3,1),
        PRIMARY KEY (essn, pno),
        FOREIGN KEY (essn) REFERENCES EMPLOYEE(ssn),
        FOREIGN KEY (pno)  REFERENCES PROJECT(pnumber)
    );
    ```

</div>

<div class="fragment">

??? info "**Step 6 — multivalued `locations`.**"
    ```sql
    CREATE TABLE DEPT_LOCATIONS (
        dnumber INT, dlocation VARCHAR(50),
        PRIMARY KEY (dnumber, dlocation),
        FOREIGN KEY (dnumber) REFERENCES DEPARTMENT(dnumber)
    );
    ```

</div>

<div class="fragment">

??? success "**Result.**"
    Seven tables: `EMPLOYEE`, `DEPARTMENT`, `PROJECT`, `DEPENDENT`, `WORKS_ON`, `DEPT_LOCATIONS`. All keys and foreign keys defined. No redundancy.

</div>

---

## 4.9 — EER Extra Steps

???+ note "Step 8 — Specialisation / Generalisation"
    Four options (Elmasri §9.2):

    | Option | Structure | When to use |
    |---|---|---|
    | **8A** | Superclass table + subclass tables (each references superclass) | Any specialisation |
    | **8B** | Subclass tables only (each contains all superclass attributes) | Total, disjoint specialisation |
    | **8C** | Single table + type discriminator | Disjoint, no subclass-specific attributes |
    | **8D** | Single table + multiple boolean type attributes | Overlapping |

<div class="fragment">

???+ note "Step 9 — Categories (Union types)"
    Create one relation per subclass, plus a **category table** that holds the primary key of each participant and a **type discriminator**. The category table's rows reference the appropriate subclass tables.

</div>

---

## Worked example 4.2 — Map EER specialisation (8A vs 8C)

!!! tip "Problem:"
    `EMPLOYEE` has subclasses `SECRETARY(typing_speed)` and `ENGINEER(engineering_field)`. Specialisation is disjoint, total. Map it two ways.

<div class="fragment">

???+ note "**Option 8A — separate tables.**"
    ```sql
    CREATE TABLE EMPLOYEE (
        ssn CHAR(9) PRIMARY KEY, first_name VARCHAR(30), last_name VARCHAR(30),
        salary DECIMAL(10,2), dno INT
    );
    CREATE TABLE SECRETARY (
        ssn CHAR(9) PRIMARY KEY,
        typing_speed INT,
        FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
    );
    CREATE TABLE ENGINEER (
        ssn CHAR(9) PRIMARY KEY,
        engineering_field VARCHAR(50),
        FOREIGN KEY (ssn) REFERENCES EMPLOYEE(ssn)
    );
    ```
    
    Every employee query must `JOIN` with the subclass table.

</div>

<div class="fragment">

??? info "**Option 8C — single table with discriminator.**"
    ```sql
    CREATE TABLE EMPLOYEE (
        ssn CHAR(9) PRIMARY KEY, first_name VARCHAR(30), last_name VARCHAR(30),
        salary DECIMAL(10,2), dno INT,
        emp_type CHAR(1) CHECK (emp_type IN ('S','E')),
        typing_speed INT,
        engineering_field VARCHAR(50)
    );
    ```
    
    Subclass-specific attributes are NULL for the "wrong" type. **Simpler** queries but **NULLs** everywhere.

</div>

<div class="fragment">

??? success "**Which to choose?**"
    - **8A** when there are **many** subclass-specific attributes and queries usually touch subclasses separately.
    - **8C** when subclass-specific attributes are **few** and queries usually join back to the superclass.

</div>

---

## 4.10 — Functional Dependencies (FDs)

???+ abstract "Definition (Elmasri §14.2)"
    A **functional dependency** $X \to Y$ on relation $R$ holds if, for every two tuples $t_1, t_2$ of $R$:
    
    $$
    t_1[X] = t_2[X] \implies t_1[Y] = t_2[Y].
    $$
    
    In words: **$X$ determines $Y$** — the value of $X$ uniquely determines the value of $Y$.

<div class="fragment">

???+ info "Example"
    - `ssn → name` — two employees with the same SSN have the same name.
    - `ssn → (name, bdate, address)` — an SSN determines the whole employee record.
    - `(ssn, pno) → hours` — the hours on a project depend on both the employee and the project.

</div>

<div class="fragment">

!!! warning "The direction is one-way"
    `X → Y` **does not** imply `Y → X`. `ssn → name` holds, but `name → ssn` does not (two employees can share a name).

</div>

---

## 4.11 — Armstrong's Axioms

???+ abstract "Three inference rules (sound and complete)"
    **Reflexivity.** If $Y \subseteq X$, then $X \to Y$.
    
    **Augmentation.** If $X \to Y$, then $XZ \to YZ$ for any $Z$.
    
    **Transitivity.** If $X \to Y$ and $Y \to Z$, then $X \to Z$.

<div class="fragment">

???+ info "Derived rules"
    - **Union:** $X \to Y$ and $X \to Z$ ⇒ $X \to YZ$.
    - **Decomposition:** $X \to YZ$ ⇒ $X \to Y$ and $X \to Z$.
    - **Pseudotransitivity:** $X \to Y$ and $WY \to Z$ ⇒ $WX \to Z$.

</div>

<div class="fragment">

!!! tip "Sound and complete"
    **Sound** = every derived FD is true in every instance satisfying the original FDs. **Complete** = every FD implied by the originals can be derived. Armstrong's axioms are both.

</div>

---

## 4.12 — Closure of Attribute Sets

???+ note "Definition"
    The **closure** $X^+$ of an attribute set $X$ under $F$ is the set of all attributes functionally determined by $X$ using Armstrong's axioms.

<div class="fragment">

???+ info "Algorithm"
    ```
    X+ := X
    repeat:
        for each FD (A → B) in F:
            if A ⊆ X+ and B ⊄ X+:
                X+ := X+ ∪ B
    until X+ does not change
    ```

</div>

<div class="fragment">

???+ success "Uses of closure"
    - **Test if $X$ is a superkey:** $X^+$ = all attributes.
    - **Test if $X \to Y$ holds:** $Y \subseteq X^+$.
    - **Compute candidate keys:** find minimal $X$ with $X^+$ = all attributes.

</div>

---

## Worked example 4.3 — Compute $(AB)^+$

!!! tip "Problem:"
    $R(A,B,C,D,E)$ with $F = \{AB \to C,\ C \to D,\ D \to E\}$. Compute $(AB)^+$.

<div class="fragment">

???+ note "**Step 1 — initialise.**"
    $(AB)^+ = \{A, B\}$.

</div>

<div class="fragment">

??? info "**Step 2 — apply FDs.**"
    - $AB \to C$: $AB \subseteq (AB)^+$, so add $C$ → $\{A,B,C\}$.
    - $C \to D$: $C \subseteq (AB)^+$, so add $D$ → $\{A,B,C,D\}$.
    - $D \to E$: $D \subseteq (AB)^+$, so add $E$ → $\{A,B,C,D,E\}$.

</div>

<div class="fragment">

??? success "**Result.**"
    $(AB)^+ = \{A, B, C, D, E\}$ — all attributes. So $AB$ is a **superkey**.
    
    **Is $AB$ a candidate key?** Check $(A)^+ = \{A\}$, $(B)^+ = \{B\}$ — neither. So $AB$ is a **candidate key** (minimal).

</div>

---

## 4.13 — Minimal Cover

???+ note "Definition"
    A **minimal cover** $F_{\min}$ of $F$ is a set of FDs equivalent to $F$ such that:
    
    1. Every FD has a **single attribute on the right**.
    2. **No redundant FD** — removing any loses equivalence.
    3. **No redundant attribute on the left** — the left side is minimal.

<div class="fragment">

???+ info "Algorithm"
    1. Decompose RHS: $X \to AB$ becomes $X \to A$, $X \to B$.
    2. Remove redundant LHS attributes: if $A$ is redundant in $AB \to C$ (i.e., $B \to C$ after removing $A$), drop $A$.
    3. Remove redundant FDs: for each FD, remove it and see if it's still implied.

</div>

<div class="fragment">

!!! tip "Minimal covers are not unique"
    Different removal orders give different but equivalent minimal covers. Any is valid.

</div>

---

## Worked example 4.4 — Compute a minimal cover

!!! tip "Problem:"
    $F = \{A \to B, ABCD \to E, EF \to G, EF \to H, ACDF \to EG\}$. Compute a minimal cover.

<div class="fragment">

???+ note "**Step 1 — decompose RHS.**"
    - $A \to B$
    - $ABCD \to E$
    - $EF \to G$, $EF \to H$
    - $ACDF \to E$, $ACDF \to G$

</div>

<div class="fragment">

??? info "**Step 2 — remove redundant LHS attributes.**"
    - For $ABCD \to E$: is $B, C, D$ redundant? Compute $(ACD)^+$. $A \to B$, so $(ACD)^+ \supseteq ABCD \to E$. So $B$ is redundant. New FD: $ACD \to E$.
    - Similarly, $C$ and $D$ may be reducible further; compute $(AD)^+ \supseteq E$? Check step by step.

</div>

<div class="fragment">

??? info "**Step 3 — remove redundant FDs.**"
    Test each FD by removing it and checking if the remainder implies it via closure.

</div>

<div class="fragment">

??? success "**Result.**"
    A possible minimal cover: $\{A \to B, A \to E, A \to G, EF \to H\}$ (depending on redundancy analysis). The exact form varies, but the closure is preserved.

</div>

---

## 4.14 — Normal Forms Overview

???+ note "The four normal forms"
    | Normal Form | Removes | Requires |
    |---|---|---|
    | **1NF** | Multivalued attributes | Atomic values |
    | **2NF** | Partial dependencies | 1NF + no partial dependency on part of a key |
    | **3NF** | Transitive dependencies | 2NF + every FD has superkey LHS **or** prime RHS |
    | **BCNF** | All non-superkey determinants | 3NF + **every** FD has superkey LHS |

<div class="fragment">

!!! tip "The hierarchy"
    BCNF $\subset$ 3NF $\subset$ 2NF $\subset$ 1NF. Every BCNF relation is 3NF, every 3NF is 2NF, every 2NF is 1NF.

</div>

---

## 4.15 — 1NF — First Normal Form

???+ abstract "1NF (Elmasri §14.3)"
    A relation is in **1NF** if every attribute value is **atomic** — no sets, no lists, no nested relations.

<div class="fragment">

???+ info "Violation example"
    ```
    STUDENT(student_id, name, phones)
    101, Alice, {555-1234, 555-5678}      ← multivalued
    ```
    
    Fix: separate table `STUDENT_PHONE(student_id, phone)`.

</div>

<div class="fragment">

!!! warning "1NF is not optional"
    Relational algebra and SQL assume atomic values. Without 1NF, projection, join, and aggregate become ill-defined.

</div>

---

## 4.16 — 2NF — Second Normal Form

???+ abstract "2NF"
    A relation is in **2NF** if it is in 1NF **and** every non-prime attribute is **fully functionally dependent** on every candidate key.

<div class="fragment">

???+ info "What is a partial dependency?"
    A **partial dependency** exists when a non-prime attribute depends on **part** of a composite candidate key, not the whole key.
    
    Example: `WORKS_ON(essn, pno, hours, ename)` with FD `essn → ename`. `ename` depends only on `essn`, part of the composite key `(essn, pno)`. **Violates 2NF.**

</div>

<div class="fragment">

???+ info "Fix"
    Decompose so that every non-prime attribute depends on the full key:
    ```sql
    EMPLOYEE(ssn, ename, ...)
    WORKS_ON(essn, pno, hours)
    ```

</div>

<div class="fragment">

!!! tip "2NF only matters for composite keys"
    If a relation has a single-attribute primary key, it is **automatically in 2NF** — no partial dependency is possible.

</div>

---

## 4.17 — 3NF — Third Normal Form

???+ abstract "3NF (Elmasri §14.4)"
    A relation is in **3NF** if it is in 2NF **and** no non-prime attribute is **transitively dependent** on any candidate key.

    Equivalently: for every non-trivial FD $X \to Y$:
    
    - $X$ is a **superkey**, **or**
    - $Y$ is a **prime attribute** (part of some candidate key).

<div class="fragment">

???+ info "What is a transitive dependency?"
    `X → Y → Z` where `X` is a key and `Z` is non-prime. `Z` depends on `X` only through `Y`.
    
    Example: `EMPLOYEE(ssn, name, dno, dname)`. `ssn → dno` (key → non-key) and `dno → dname` (non-key → non-key). `dname` is transitively dependent on `ssn`. **Violates 3NF.**

</div>

<div class="fragment">

???+ info "Fix"
    Decompose into `EMPLOYEE(ssn, name, dno)` and `DEPARTMENT(dno, dname)`.

</div>

---

## 4.18 — BCNF — Boyce-Codd Normal Form

???+ abstract "BCNF"
    A relation is in **BCNF** if, for every non-trivial FD $X \to Y$:
    
    **$X$ is a superkey.**
    
    (No exception for prime attributes.)

<div class="fragment">

???+ info "BCNF is stricter than 3NF"
    3NF allows $Y$ prime even when $X$ is not superkey. BCNF forbids this.
    
    Example: `R(A,B,C)` with `AB → C, C → B`. Candidate keys: `AB`, `AC`. `C → B` has non-superkey `C` but prime `B` → **3NF but not BCNF**.

</div>

<div class="fragment">

!!! warning "BCNF may lose dependency preservation"
    The classic 3NF-but-not-BCNF relation above cannot be BCNF-decomposed without losing the FD `AB → C`. So 3NF is the practical target when dependency preservation matters.

</div>

---

## Worked example 4.5 — Full normalisation

!!! tip "Problem:"
    ```
    EMP_PROJ(ssn, ename, pno, pname, hours)
    ```
    FDs: `ssn → ename`, `pno → pname`, `(ssn, pno) → hours`. Normalise to 3NF.

<div class="fragment">

???+ note "**Step 1 — candidate key.**"
    Only `(ssn, pno)` is a candidate key. Prime attributes: `ssn`, `pno`. Non-prime: `ename`, `pname`, `hours`.

</div>

<div class="fragment">

??? info "**Step 2 — check 1NF.**"
    All values atomic. **Satisfied.**

</div>

<div class="fragment">

??? info "**Step 3 — check 2NF.**"
    `ssn → ename`: partial dependency (ename on ssn only). `pno → pname`: partial dependency. **Violates 2NF.**

</div>

<div class="fragment">

??? info "**Step 4 — decompose for 2NF.**"
    - `EMP(ssn, ename)`
    - `PROJ(pno, pname)`
    - `WORKS_ON(ssn, pno, hours)`

</div>

<div class="fragment">

??? info "**Step 5 — check 3NF.**"
    - `EMP`: `ssn` is the key; `ssn → ename` OK.
    - `PROJ`: `pno` is the key; `pno → pname` OK.
    - `WORKS_ON`: `(ssn, pno)` is the key; `(ssn, pno) → hours` OK. No transitive dependencies.
    
    **All three are in 3NF (and in BCNF).**

</div>

<div class="fragment">

??? success "**Result.**"
    `EMP(ssn, ename)`, `PROJ(pno, pname)`, `WORKS_ON(ssn, pno, hours)` — lossless, dependency-preserving, in BCNF.

</div>

---

## Worked example 4.6 — 3NF but not BCNF

!!! tip "Problem:"
    `R(A, B, C)` with FDs `AB → C, C → B`. Is it in 3NF? BCNF?

<div class="fragment">

???+ note "**Step 1 — candidate keys.**"
    - $(AB)^+ = \{A, B, C\}$ → `AB` is a candidate key.
    - $(AC)^+ = \{A, C, B\}$ → `AC` is a candidate key.
    
    Prime attributes: `A, B, C`. Non-prime: none.

</div>

<div class="fragment">

??? info "**Step 2 — check 3NF.**"
    For every FD $X \to Y$: $X$ superkey **or** $Y$ prime.
    - `AB → C`: `AB` is superkey. ✓
    - `C → B`: `C` not superkey, but `B` is prime → satisfies 3NF. ✓

</div>

<div class="fragment">

??? info "**Step 3 — check BCNF.**"
    `C → B`: `C` is not a superkey. **Violates BCNF.**

</div>

<div class="fragment">

??? success "**Conclusion.**"
    The relation is **3NF but not BCNF**.

</div>

<div class="fragment">

???+ warning "BCNF decomposition loses dependency"
    Decompose into `R1(C, B)` and `R2(A, C)`. Lossless, BCNF. But `AB → C` now requires a **join** to verify — **dependency not preserved**.

</div>

---

## 4.19 — Decomposition Properties

???+ note "Two desirable properties"
    | Property | Meaning |
    |---|---|
    | **Lossless join** | The natural join of the decomposed relations reconstructs the original exactly (no spurious tuples) |
    | **Dependency preservation** | Every FD of the original is enforceable within a single decomposed relation |

<div class="fragment">

???+ info "Testing for lossless join (binary decomposition)"
    For $R$ decomposed into $R_1$ and $R_2$:
    
    **Lossless iff** $R_1 \cap R_2 \to R_1$ **or** $R_1 \cap R_2 \to R_2$ holds.

</div>

<div class="fragment">

???+ info "The 3NF synthesis algorithm"
    1. Compute a minimal cover $F_{\min}$.
    2. For each FD $X \to A$ in $F_{\min}$, create a relation $XA$.
    3. If no relation contains a candidate key of $R$, add one.
    
    **Guarantees:** lossless, dependency-preserving, 3NF.

</div>

<div class="fragment">

???+ warning "BCNF decomposition is lossless but not dependency-preserving"
    The classic 3NF-not-BCNF example shows that BCNF and dependency preservation can conflict. **3NF is the practical target** for OLTP systems.

</div>

---

## 4.20 — Informal Design Guidelines

???+ note "Three guidelines (Elmasri §14.1)"
    **1. Clear semantics.** Each tuple should represent **one entity or relationship**, and each attribute should describe **one property** of that entity.

<div class="fragment">

???+ note "2. No redundant information"
    Redundancy causes **update, insertion, and deletion anomalies**. The goal of normalization is to eliminate structural redundancy.

</div>

<div class="fragment">

???+ note "3. Few NULLs"
    Excessive NULLs waste space and complicate joins, aggregates, and constraints. NULLs also indicate **ambiguous design**.

</div>

<div class="fragment">

!!! tip "Normalization is a design guideline, not a law"
    Sometimes denormalization is **deliberate** — data warehouses denormalize for query performance. But OLTP systems should normalize to 3NF (or BCNF when possible).

</div>

---

## Chapter 4 — key takeaways

<div class="fragment">

**1.** The **7-step ER-to-relational mapping** transforms any ER diagram into a relational schema. Order matters.

</div>

<div class="fragment">

**2.** **EER specialisations** have four mapping options (8A–8D); **categories** use a type-discriminator table.

</div>

<div class="fragment">

**3.** **Functional dependencies** $X \to Y$ mean $X$ determines $Y$. **Armstrong's axioms** (reflexivity, augmentation, transitivity) are sound and complete.

</div>

<div class="fragment">

**4.** **Closure** $X^+$ tests whether $X$ is a superkey and whether $X \to Y$ holds. **Minimal cover** reduces FDs to their essential form.

</div>

<div class="fragment">

**5.** **1NF** = atomic values. **2NF** = no partial dependency. **3NF** = no transitive dependency. **BCNF** = every determinant is a superkey.

</div>

<div class="fragment">

**6.** **Lossless join** and **dependency preservation** are the two goals of decomposition. **3NF synthesis** achieves both; **BCNF decomposition** achieves only losslessness.

</div>

---

## Next — Chapter 5

**Relational Algebra and Calculus.**

<div class="fragment">

- **σ, π, ×, ∪, −, ρ** — the six fundamental operations.
- **Joins**: theta, equi, natural, outer.
- **Division** — for "for all" queries.
- **Tuple relational calculus (TRC)** and **domain relational calculus (DRC)**.
- **Examples** on the COMPANY database.

</div>

<div class="fragment">

Chapter 4 gave a normalized schema. Chapter 5 gives the **query algebra** — the theoretical foundation of SQL.

</div>