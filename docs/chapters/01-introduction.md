# Chapter 1 — Introduction

!!! abstract "Chapter at a glance"
    **Contact hours:** 2 lecture + 1 tutorial · **Home study:** ~5 h

    **Coverage:** manual file handling systems · traditional file-based systems · database approach · DBMS · components of the DBMS environment · database development life cycle · common uses · ANSI-SPARC architecture · multi-user DBMS.

---

## Learning outcomes

1. Explain the limitations of file-based systems that motivate the database approach.
2. Describe the components of a DBMS environment.
3. Draw the ANSI-SPARC three-schema architecture and explain data independence.
4. List the phases of the database development life cycle (DBLC).

---

## 1.1 File-based systems and their limitations

Before DBMSs, data was stored in separate files managed by application programs.

**Limitations (Elmasri §1.2):**

| Limitation | Description |
|---|---|
| **Data redundancy** | Same data stored in multiple files |
| **Data inconsistency** | Different copies diverge over time |
| **Program-data dependence** | File format defined in the program — changing the format requires changing every program |
| **Difficulty of access** | New queries require new programs |
| **Data isolation** | Data scattered across files in different formats |
| **Integrity problems** | Constraints enforced only in programs, not in data |
| **Atomicity problems** | A crash can leave the data half-updated |
| **Concurrent-access anomalies** | Multiple users can corrupt data if not coordinated |
| **Security problems** | Access control at file level only |

**Example.** A university has separate files for students, courses, and enrolments. A student's name may appear in all three; updating it in one file but not the others creates inconsistency.

---

## 1.2 The database approach

!!! note "Definition (Elmasri §1.3)"
    A **database** is an organised collection of structured data. A **DBMS** is software that manages and controls access to the database.

The database approach has these properties:

- **Self-describing:** the database stores its own schema (catalog or data dictionary).
- **Insulation:** programs are independent of data format (program–data independence).
- **Multiple views:** different users see different subsets.
- **Sharing:** multiple users and programs share one data store.
- **Transaction support:** atomicity, consistency, isolation, durability (ACID).
- **Integrity constraints:** enforced by the DBMS.

---

## 1.3 Components of a DBMS environment

```
+-----------------------------------------------+
|  Users: end users, application programmers, DBA |
+-----------------------------------------------+
|  Application programs and queries              |
+-----------------------------------------------+
|  DBMS: query processor, storage manager         |
+-----------------------------------------------+
|  Stored data                                    |
+-----------------------------------------------+
```

**Five components (Elmasri §1.5):**

1. **Hardware** — physical storage.
2. **Software** — the DBMS itself, plus application programs.
3. **Data** — the database and the catalog.
4. **Procedures** — instructions for using the system.
5. **People** — end users, application programmers, DBAs, database designers.

---

## 1.4 ANSI-SPARC three-schema architecture

!!! note "ANSI-SPARC (Elmasri §2.2)"
    The three-level architecture separates:

    1. **External (view) level** — how users see the data.
    2. **Conceptual (logical) level** — the community-wide logical structure.
    3. **Internal (physical) level** — how data is stored.

```
Users ─► External schemas (views)
                │
                ▼  logical data independence
        Conceptual schema (community view)
                │
                ▼  physical data independence
        Internal schema (storage details)
                │
                ▼
           Stored database
```

**Data independence:**

- **Logical data independence** — change the conceptual schema without changing the external schemas.
- **Physical data independence** — change the internal schema without changing the conceptual schema.

---

## 1.5 The database development life cycle (DBLC)

| Phase | Description |
|---|---|
| **1. Requirements analysis** | Interview users, produce a requirements specification |
| **2. Conceptual design** | Build the ER/EER model, independent of any DBMS |
| **3. Choice of DBMS** | Evaluate vendors against the requirements |
| **4. Logical design** | Map the conceptual model to a relational schema; normalise |
| **5. Physical design** | Choose file structures, indexes, storage layout |
| **6. Implementation** | Create the schema in the DBMS, load data |
| **7. Testing** | Validate against requirements |
| **8. Operation and maintenance** | Monitor, tune, evolve |

The DBLC is **iterative** — discovery in later phases often forces refinement of earlier ones.

---

## 1.6 Multi-user DBMS

A **multi-user DBMS** supports concurrent access by many users. Key concerns:

- **Concurrency control** — prevent simultaneous updates from corrupting data.
- **Transaction management** — atomic commits, rollback on failure.
- **Recovery** — restore a consistent state after a crash.
- **Security** — authenticate users, authorise access.
- **Views** — present tailored subsets of the data.

---

## 1.7 Quiz

1. List five limitations of file-based systems.
2. What is a DBMS? Name four of its key features.
3. Draw the ANSI-SPARC three-schema architecture and explain each level.
4. Distinguish logical from physical data independence.
5. List the phases of the database development life cycle.

---

## Answer key

??? success "Click to reveal"

    1. Data redundancy, inconsistency, program-data dependence, difficulty of access, concurrent-access anomalies.
    2. A DBMS is software that manages and controls access to a database. Key features: self-describing, program-data independence, multiple views, ACID transactions.
    3. (See diagram above.)
    4. Logical: change conceptual schema without changing external views. Physical: change internal schema without changing conceptual schema.
    5. Requirements analysis, conceptual design, choice of DBMS, logical design, physical design, implementation, testing, operation and maintenance.

---

## Further reading

- Elmasri & Navathe, §1.1–§1.8, §2.1–§2.3.
- Silberschatz et al., §1.1–§1.4.
- Ramakrishnan & Gehrke, §1.1–§1.3.