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
  js: assets/lecture-03.js

---

# ECEG-4191

## Chapter 3 — Conceptual Design and E-R Modelling

---

## Learning outcomes

<div class="fragment">

1. Describe the **three phases of database design** and where conceptual design fits.

</div>

<div class="fragment">

2. Draw ER diagrams with **entities**, **attributes**, **relationships**, **cardinality ratios**, and **participation constraints**.

</div>

<div class="fragment">

3. Model **weak entities** with identifying relationships and partial keys.

</div>

<div class="fragment">

4. Apply **EER concepts**: subclasses, superclasses, specialisation, generalisation, categories.

</div>

<div class="fragment">

5. Build a complete ER diagram for the **COMPANY** database.

</div>

---

## 3.1 — The Three Design Phases

???+ note "Elmasri §3.1"
    | Phase | Input | Output |
    |---|---|---|
    | **Conceptual design** | Requirements | ER / EER model |
    | **Logical design** | Conceptual model | Relational schema |
    | **Physical design** | Logical schema | File structures, indexes |

<div class="fragment">

!!! tip "Why conceptual design matters most"
    Errors in conceptual design **propagate everywhere**. A wrong entity or relationship cannot be fixed in later phases without redesigning. Conceptual design is also the phase where **communication with users** happens.

</div>

<div class="fragment">

???+ warning "Conceptual design is DBMS-independent"
    The conceptual model must **not** mention tables, indexes, or SQL. Those belong to later phases. The ER model describes **what data means**, not how it will be stored.

</div>

---

## 3.2 — The ER Model Components

???+ note "Three building blocks"
    | Component | Purpose | Example |
    |---|---|---|
    | **Entity type** | A category of distinguishable things | `EMPLOYEE`, `PROJECT` |
    | **Attribute** | A property of an entity type | `name`, `salary` |
    | **Relationship type** | An association between two entity types | `WORKS_FOR`, `MANAGES` |

<div class="fragment">

???+ info "Instance vs type"
    - **Entity type** — the category (`EMPLOYEE`).
    - **Entity** — an individual (`Alice`).
    - **Relationship type** — the association (`WORKS_FOR`).
    - **Relationship instance** — one specific link (`Alice WORKS_FOR Sales`).

</div>

---

## 3.3 — Attribute Types

???+ note "Six kinds of attributes"
    | Type | Definition | Notation |
    |---|---|---|
    | **Simple** | Single atomic value | Ellipse |
    | **Composite** | Composed of sub-attributes | Ellipse with sub-ellipses |
    | **Multivalued** | Multiple values per entity | Double ellipse |
    | **Derived** | Computed from other attributes | Dashed ellipse |
    | **Key** | Uniquely identifies | Underlined |
    | **Partial key** | Identifies within an owner's set (weak entities) | Dashed underline |

<div class="fragment">

???+ info "Examples"
    - `address` — composite (street, city, zip).
    - `phone` — multivalued (a person has many).
    - `age` — derived (from `birth_date`).
    - `ssn` — key.
    - `dep_name` — partial key of a weak `DEPENDENT`.

</div>

<div class="fragment">

!!! tip "When to model a multivalued attribute vs. a new entity"
    If the values are **simple** (a phone number, a city), use a multivalued attribute. If they have their **own attributes** (a university degree with `year`, `major`, `GPA`), create a separate entity type. Attribute = "property of", entity = "independent existence".

</div>

---

## 3.4 — Entity and Relationship Notation

???+ note "Standard ER diagram notation"
    | Symbol | Meaning |
    |---|---|
    | Rectangle | Entity type |
    | **Double rectangle** | Weak entity type |
    | Ellipse | Attribute |
    | Underlined attribute | Key |
    | Dashed ellipse | Derived attribute |
    | Double ellipse | Multivalued attribute |
    | Diamond | Relationship type |
    | **Double diamond** | Identifying relationship (weak) |
    | Line | Participation link |
    | **Double line** | Total participation |
    | Label `1`, `N`, `M` | Cardinality ratio |

---

## 3.5 — Cardinality Ratios

???+ note "Three ratios"
    For a relationship between entity types $A$ and $B$:

    | Ratio | Meaning |
    |---|---|
    | **1:1** | One $A$ relates to at most one $B$, and vice versa |
    | **1:N** | One $A$ relates to many $B$, but each $B$ relates to one $A$ |
    | **M:N** | Many $A$ relate to many $B$ |

<div class="fragment">

???+ info "Real-world examples"
    - **1:1** — `DEPARTMENT` `MANAGES` `EMPLOYEE` (one dept has one manager; one manager manages one dept).
    - **1:N** — `DEPARTMENT` `CONTROLS` `PROJECT` (one dept controls many projects; each project belongs to one dept).
    - **M:N** — `EMPLOYEE` `WORKS_ON` `PROJECT` (an employee works on many projects; a project has many employees).

</div>

<div class="fragment">

!!! warning "Cardinality is about the association, not the entities"
    The 1:N in `DEPARTMENT CONTROLS PROJECT` means *one dept controls many projects*. It does not say "a project has many employees" — that's a different relationship.

</div>

---

## 3.6 — Participation Constraints

???+ note "Total vs partial"
    - **Total participation** (double line) — every entity of the type **must** participate.
    - **Partial participation** (single line) — some entities may not participate.

<div class="fragment">

???+ info "Examples in COMPANY"
    | Relationship | Participation |
    |---|---|
    | `EMPLOYEE WORKS_FOR DEPARTMENT` | **Total** on EMPLOYEE (every employee works for a department) |
    | `EMPLOYEE MANAGES DEPARTMENT` | **Partial** on EMPLOYEE (not every employee manages) |
    | `PROJECT CONTROLLED_BY DEPARTMENT` | **Total** on PROJECT (every project belongs to a department) |

</div>

<div class="fragment">

!!! tip "Participation determines NOT NULL in the schema"
    Total participation on the "N" side becomes **NOT NULL** in the relational schema (Chapter 4). Partial participation allows NULL.

</div>

---

## Worked example 3.1 — A simple ER diagram

!!! tip "Problem:"
    Model a small library:
    - Books have an ISBN, title, and year.
    - Members have a member_id and name.
    - A member can borrow books; each loan records the borrow date.

<div class="fragment">

???+ note "**Step 1 — identify entities.**"
    - `BOOK` (ISBN, title, year).
    - `MEMBER` (member_id, name).
    - `LOAN` — records a borrowing event.

</div>

<div class="fragment">

??? info "**Step 2 — decide: entity or relationship?**"
    `LOAN` could be a **relationship** (BOOK-MEMBER) with attribute `borrow_date`. But if we want to track multiple loans over time, `LOAN` should be an **entity**.

</div>

<div class="fragment">

??? info "**Step 3 — relationships.**"
    - `BOOK` — `LOAN` — `MEMBER` with cardinality 1:N and M:1 (a book can be loaned many times; a member can borrow many times).

</div>

<div class="fragment">

??? success "**Step 4 — the diagram.**"
    ```
    MEMBER(1) ──< LOAN >── (1)BOOK
                  |
               borrow_date
    ```
    
    Actually, the correct model is: MEMBER 1 — N LOAN, BOOK 1 — N LOAN. Each loan ties one book to one member.

</div>

---

## 3.7 — Weak Entities

???+ note "Definition (Elmasri §3.5)"
    A **weak entity type** has no key attribute of its own and depends on an **owner entity type** through an **identifying relationship**.

<div class="fragment">

???+ info "Key characteristics"
    - **No key of its own.**
    - Has a **partial key** — an attribute that is unique only within one owner's set.
    - The full key = (owner's key, partial key).
    - Shown as a **double rectangle** with a **double diamond** identifying relationship.

</div>

<div class="fragment">

???+ note "Example: `DEPENDENT`"
    - `DEPENDENT` (of an `EMPLOYEE`) has `dep_name`, `sex`, `birth_date`, `relationship`.
    - `dep_name` is unique only **within one employee**.
    - Full key: `(essn, dep_name)`.
    - Cannot exist without an employee.

</div>

---

## Worked example 3.2 — Model with a weak entity

!!! tip "Problem:"
    A company has departments. Each department has projects. Each project has parts. A part is identified by a part_number that is unique only within its project.

<div class="fragment">

???+ note "**Step 1 — regular entities.**"
    `DEPARTMENT(dnumber, dname)`, `PROJECT(pnumber, pname)`.

</div>

<div class="fragment">

??? info "**Step 2 — the weak entity.**"
    `PART(part_number, name, weight)` — `part_number` unique only within a project.

</div>

<div class="fragment">

??? info "**Step 3 — identifying relationship.**"
    `PART` **has** `PROJECT` (double diamond). Full key of `PART` = `(pnumber, part_number)`.

</div>

<div class="fragment">

??? success "**Step 4 — the diagram.**"
    ```
    DEPARTMENT(1) ──< PROJECT >── 1
                        ||||
                        └────< PART > (weak, partial key: part_number)
    ```

</div>

---

## 3.8 — EER Concepts: Overview

???+ note "Enhanced ER adds four constructs"
    | Construct | Purpose |
    |---|---|
    | **Subclass / Superclass** | Classification hierarchy |
    | **Specialisation** | Top-down decomposition |
    | **Generalisation** | Bottom-up combination |
    | **Category (Union)** | Entity belongs to one of several superclasses |

<div class="fragment">

???+ info "Why EER?"
    The basic ER model cannot express **is-a** relationships elegantly. EER adds inheritance — subclasses inherit the superclass's attributes and relationships.

</div>

---

## 3.9 — Subclass and Superclass

???+ note "Definition"
    A **subclass** is a subset of a **superclass**'s entities. Each subclass member is also a superclass member (is-a relationship).

<div class="fragment">

???+ info "Example"
    - `EMPLOYEE` is the superclass.
    - `SECRETARY`, `ENGINEER`, `TECHNICIAN` are subclasses.
    - Every engineer **is an** employee; not every employee is an engineer.

</div>

<div class="fragment">

???+ note "Attribute inheritance"
    Subclasses inherit **all** attributes and relationship participations of their superclass — including inherited keys.

</div>

<div class="fragment">

!!! tip "When to model a subclass"
    Use a subclass when:
    - It has **its own attributes** (e.g., `ENGINEER` has `engineering_field`).
    - It has **its own relationships** (e.g., `ENGINEER` belongs to a `PROFESSIONAL_UNION`).
    - It has **its own participation** in a shared relationship.

</div>

---

## 3.10 — Specialisation and Generalisation

???+ note "Two directions, same diagram"
    | Process | Direction | Starting point |
    |---|---|---|
    | **Specialisation** | Top-down | Start with one entity, identify subclasses |
    | **Generalisation** | Bottom-up | Start with several entities, combine into superclass |

<div class="fragment">

???+ info "Example: specialisation"
    Start with `EMPLOYEE`. Specialise into `SECRETARY`, `ENGINEER`, `TECHNICIAN` based on job function.

</div>

<div class="fragment">

???+ info "Example: generalisation"
    Start with `CAR` and `TRUCK`. Generalise into `VEHICLE` (they share `price`, `max_speed`, `registration`).

</div>

<div class="fragment">

!!! tip "The diagram is the same"
    Both produce a superclass with a circle (or triangle) connecting to subclasses. Only the **direction of reasoning** differs — which affects the modelling conversation, not the notation.

</div>

---

## 3.11 — Constraints on Specialisation

???+ note "Two dimensions"
    | Dimension | Values | Meaning |
    |---|---|---|
    | **Disjointness** | Disjoint (d) | An entity belongs to at most one subclass |
    |  | Overlapping (o) | An entity may belong to multiple subclasses |
    | **Completeness** | Total (double line) | Every superclass entity belongs to some subclass |
    |  | Partial (single line) | Some superclass entities belong to no subclass |

<div class="fragment">

???+ info "The four combinations"
    | Disjoint | Complete | Example |
    |---|---|---|
    | **D, Total** | every employee is exactly one of secretary/engineer/technician |
    | **D, Partial** | some employees are unclassified |
    | **O, Total** | every employee is at least one of manager/engineer (some are both) |
    | **O, Partial** | some employees are neither manager nor engineer |

</div>

<div class="fragment">

!!! warning "Common mistake"
    Disjointness is a **constraint of the model**, not a claim about the world. If an employee can be both an engineer and a manager, the specialisation must be **overlapping**.

</div>

---

## Worked example 3.3 — EER for a university

!!! tip "Problem:"
    Model:
    - `PERSON(pid, name, address)`.
    - A person can be a `STUDENT` (with `major`) and/or a `FACULTY` (with `rank`).
    - A `STUDENT` can be `GRAD_STUDENT` (with `thesis_title`) or `UNDERGRAD_STUDENT` (with `year`).

<div class="fragment">

???+ note "**Step 1 — superclass.**"
    `PERSON`.

</div>

<div class="fragment">

??? info "**Step 2 — first-level specialisation.**"
    `STUDENT` and `FACULTY`. **Overlapping**, **partial** — a person may be neither, and may be both.

</div>

<div class="fragment">

??? info "**Step 3 — second-level specialisation.**"
    `GRAD_STUDENT` and `UNDERGRAD_STUDENT` under `STUDENT`. **Disjoint**, **total** — a student is exactly one.

</div>

<div class="fragment">

??? success "**Step 4 — the diagram.**"
    ```
                  PERSON
                   /  \
              STUDENT  FACULTY
                 / \   (overlapping, partial)
                /   \
       GRAD_STUDENT  UNDERGRAD_STUDENT
       (disjoint, total)
    ```

</div>

<aside class="notes">
  <div style="font-size: 16px; line-height: 1.6;">
Walk through why the first-level specialisation must be overlapping: a graduate teaching assistant is both a student and faculty. If we marked it disjoint, the model would forbid this common case. This is a favourite exam question — the disjoint/total choices must be justified from the requirements, not assumed.
  </div>
</aside>

---

## 3.12 — Categories (Union Types)

???+ note "Definition"
    A **category** (or **union type**) combines entities from **disjoint** superclasses. An entity belongs to **exactly one** of the superclasses.

<div class="fragment">

???+ info "Example"
    An `OWNER` may be a `PERSON`, a `BANK`, or a `COMPANY` — **mutually exclusive** alternatives.
    
    ```
    PERSON    BANK    COMPANY
       \       |       /
        \      |      /
         \     |     /
          ◯ ∪ ◯
            |
          OWNER
    ```

</div>

<div class="fragment">

???+ info "Category vs specialisation"
    | Aspect | Category (union) | Specialisation |
    |---|---|---|
    | Direction | Subclass → multiple superclasses | Superclass → subclasses |
    | Mutually exclusive? | **Yes** | Depends on disjoint/overlapping |
    | Member of one superclass? | **Yes, exactly one** | No |

</div>

---

## Worked example 3.4 — Category in a vehicle registration system

!!! tip "Problem:"
    Vehicles can be registered. Owners can be:
    - Individuals (`PERSON`).
    - Companies (`COMPANY`).
    - Government agencies (`GOV_AGENCY`).
    
    Each owner is exactly one of these — never a combination.

<div class="fragment">

???+ note "**Step 1 — why category, not specialisation?**"
    Because a `PERSON`, a `COMPANY`, and a `GOV_AGENCY` are **disjoint** types — an entity cannot be a person and a company at the same time. Specialisation would imply a common superclass; here there is no natural common superclass, only a union.

</div>

<div class="fragment">

??? info "**Step 2 — the diagram.**"
    ```
    PERSON    COMPANY    GOV_AGENCY
       \          |          /
        \         |         /
         ◯─────∪─────◯
              |
            OWNER  (has category membership)
              |
          REGISTERED_VEHICLE
    ```

</div>

<div class="fragment">

??? success "**Step 3 — the rule.**"
    Each `OWNER` is a member of **exactly one** of the three superclasses. The category declares "these three are alternatives for the role of owner".

</div>

---

## 3.13 — The COMPANY Case Study

???+ note "Requirements"
    - **Departments** have a unique name, a number, and one employee who manages them.
    - Each department controls several **projects**, each with a unique name, number, and single location.
    - **Employees** have a name, SSN, sex, address, salary, birth date, and belong to a department.
    - Employees may work on several projects (hours per week).
    - Employees may have **dependents** (name, sex, birth date, relationship).

---

## Worked example 3.5 — COMPANY ER diagram

!!! tip "Problem:"
    Draw the complete ER diagram for COMPANY.

<div class="fragment">

???+ note "**Step 1 — entities.**"
    `EMPLOYEE`, `DEPARTMENT`, `PROJECT`, `DEPENDENT` (weak).

</div>

<div class="fragment">

??? info "**Step 2 — relationships.**"
    - `EMPLOYEE WORKS_FOR DEPARTMENT` (N:1, total on EMPLOYEE).
    - `EMPLOYEE MANAGES DEPARTMENT` (1:1, partial on both).
    - `DEPARTMENT CONTROLS PROJECT` (1:N, total on PROJECT).
    - `EMPLOYEE WORKS_ON PROJECT` (M:N, with `hours`).
    - `EMPLOYEE DEPENDENT_OF DEPENDENT` (1:N, identifying).
    - `EMPLOYEE SUPERVISES EMPLOYEE` (1:N, recursive).

</div>

<div class="fragment">

??? info "**Step 3 — attributes.**"
    - `DEPARTMENT`: `dnumber` (key), `dname` (unique), `locations` (multivalued).
    - `PROJECT`: `pnumber` (key), `pname`, `location`.
    - `EMPLOYEE`: `ssn` (key), `name` (composite: first, middle, last), `sex`, `address`, `salary`, `bdate`.
    - `DEPENDENT`: `dep_name` (partial key), `sex`, `bdate`, `relationship`.

</div>

<div class="fragment">

??? success "**Step 4 — cardinalities and participation.**"
    | Relationship | Ratio | Participation |
    |---|---|---|
    | `WORKS_FOR` | N:1 | Total on EMPLOYEE |
    | `MANAGES` | 1:1 | Partial on both |
    | `CONTROLS` | 1:N | Total on PROJECT |
    | `WORKS_ON` | M:N | Partial on both |
    | `DEPENDENT_OF` | 1:N | Total on DEPENDENT |

</div>

---

## 3.14 — COMPANY EER Extension

???+ note "Specialisation of EMPLOYEE"
    Three subclasses based on job function:
    - `SECRETARY` — with `typing_speed`.
    - `ENGINEER` — with `engineering_field`.
    - `TECHNICIAN` — with `skill_level`.

<div class="fragment">

???+ info "Constraints"
    - **Disjoint** — a secretary is not an engineer.
    - **Total** — every employee is one of the three (in this simplified model).

</div>

<div class="fragment">

???+ note "Second specialisation"
    `HOURLY_EMPLOYEE` and `SALARIED_EMPLOYEE` (disjoint, total).

</div>

<div class="fragment">

!!! tip "Multiple specialisations"
    An entity type can be specialised in **more than one way** — `EMPLOYEE` can be (a) secretary/engineer/technician and (b) hourly/salaried. These are independent dimensions.

</div>

---

## 3.15 — Common Modelling Pitfalls

???+ warning "Pitfall 1 — modelling a relationship as an entity"
    `MARRIAGE` between two `PERSON`s has its own attributes (`date`, `location`). Model it as an **entity** with two relationships to `PERSON`, not as a plain relationship.

<div class="fragment">

??? warning "Pitfall 2 — entity with only one attribute as key"
    A `PHONE(phone_number)` with only `phone_number` as attribute is really an attribute of `PERSON` (multivalued). Don't create entities without their own meaningful attributes.

</div>

<div class="fragment">

??? warning "Pitfall 3 — modelling time as a relationship"
    If "who was manager when" matters, model `MANAGEMENT(employee, department, start_date, end_date)` as an **entity** — a relationship has no notion of time.

</div>

<div class="fragment">

??? warning "Pitfall 4 — confusing subclass and category"
    Subclass = **is-a** (every engineer is an employee). Category = **is one of** (a vehicle owner is a person OR a company, not both).

</div>

---

## Chapter 3 — key takeaways

<div class="fragment">

**1.** Conceptual design comes first and errors here propagate. The conceptual model is DBMS-independent.

</div>

<div class="fragment">

**2.** Three building blocks: **entities**, **attributes** (simple, composite, multivalued, derived, key), **relationships**.

</div>

<div class="fragment">

**3.** **Cardinality ratios** (1:1, 1:N, M:N) and **participation** (total, partial) fully specify a relationship.

</div>

<div class="fragment">

**4.** **Weak entities** have no key of their own; their key = (owner's key, partial key).

</div>

<div class="fragment">

**5.** **EER** adds subclass/superclass, specialisation/generalisation, and categories — with **disjoint/overlapping** and **total/partial** constraints.

</div>

<div class="fragment">

**6.** Model entities — not relationships — when the "relationship" has its own attributes or changes over time.

</div>

---

## Next — Chapter 4

**Logical Design and Normalisation.**

<div class="fragment">

- The 7-step **ER-to-relational mapping**.
- The 2 extra steps for EER constructs.
- **Functional dependencies** and **Armstrong's axioms**.
- Closure, minimal cover.
- **Normal forms 1NF, 2NF, 3NF, BCNF**.

</div>

<div class="fragment">

Chapter 3 gave a beautiful conceptual model. Chapter 4 turns it into **tables** — and proves that the tables are well-designed.

</div>