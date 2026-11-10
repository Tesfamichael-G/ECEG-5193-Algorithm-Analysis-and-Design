# Chapter 5 — Relational Algebra and Calculus

!!! abstract "Chapter at a glance"
    **Coverage:** unary and binary relational-algebra operations · set-theoretic operations · joins · division · tuple and domain calculus · COMPANY examples.

---

## Learning outcomes

1. Express queries in the fundamental relational-algebra operations.
2. Use joins, outer joins, and division correctly.
3. Translate between relational algebra, tuple relational calculus, and domain relational calculus.

---

## 5.1 Relational algebra

!!! note "Definition (Elmasri §6.1)"
    **Relational algebra** is a procedural query language: each query is a sequence of operations on relations, producing a new relation.

**Six fundamental operations:** $\sigma, \pi, \cup, -, \times, \rho$.

### 5.1.1 Unary operations

- **Selection** $\sigma_{\text{condition}}(R)$ — keeps tuples satisfying the condition.
- **Projection** $\pi_{A_1, \ldots, A_k}(R)$ — keeps only the listed attributes (duplicates removed).
- **Rename** $\rho_{S}(R)$ — renames a relation or its attributes.

### 5.1.2 Set-theoretic operations

For $R$ and $S$ with the same schema (union-compatible):

- **Union** $R \cup S$.
- **Intersection** $R \cap S$.
- **Set difference** $R - S$.

### 5.1.3 Cartesian product

$R \times S$ — every tuple of $R$ paired with every tuple of $S$.

### 5.1.4 Joins

| Join | Symbol | Meaning |
|---|---|---|
| **Theta join** | $R \bowtie_\theta S$ | Cartesian product filtered by $\theta$ |
| **Equijoin** | $R \bowtie_{A=B} S$ | Theta join with equality |
| **Natural join** | $R \bowtie S$ | Equijoin on all common attributes, duplicate columns removed |
| **Left outer join** | $R ⟕ S$ | Keeps unmatched tuples from $R$, pads $S$ with NULLs |
| **Right outer join** | $R ⟖ S$ | Symmetric |
| **Full outer join** | $R ⟗ S$ | Union of both |

### 5.1.5 Division

$R \div S$ (with $S$'s attributes a subset of $R$'s) returns the tuples $t$ of $R$ such that for every tuple $s$ in $S$, the combined tuple $(t, s)$ is in $R$. Used for "for all" queries.

---

## Worked example 5.1 — COMPANY queries in relational algebra

**Schema.**

- `EMPLOYEE(ssn, name, salary, dno)`
- `DEPARTMENT(dnumber, dname, mgr_ssn)`
- `PROJECT(pnumber, pname, dnum)`
- `WORKS_ON(essn, pno, hours)`

**Q1.** Names of employees who work on the 'ProductX' project.

$$
\pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie \text{WORKS\_ON} \bowtie \pi_{\text{pnumber}}\bigl(\sigma_{\text{pname}=\text{'ProductX'}}(\text{PROJECT})\bigr)\bigr)
$$

**Q2.** Names of employees who do **not** work on any project.

$$
\pi_{\text{name}}\bigl(\text{EMPLOYEE}\bigr) - \pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie \text{WORKS\_ON}\bigr)
$$

**Q3.** Names of employees who work on **all** projects.

$$
\pi_{\text{name}}\Bigl(\text{EMPLOYEE} \bowtie \bigl(\pi_{\text{essn, pno}}(\text{WORKS\_ON}) \div \pi_{\text{pnumber}}(\text{PROJECT})\bigr)\Bigr)
$$

---

## Worked example 5.2 — tuple relational calculus

**Query (Q1 above) in TRC:**

$$
\{ e.\text{name} \mid \text{EMPLOYEE}(e) \land \exists\, w \in \text{WORKS\_ON} \;\exists\, p \in \text{PROJECT} \; (w.\text{essn} = e.\text{ssn} \land w.\text{pno} = p.\text{pnumber} \land p.\text{pname} = \text{'ProductX'}) \}
$$

**Query (Q3 above) in TRC:**

$$
\{ e.\text{name} \mid \text{EMPLOYEE}(e) \land \forall\, p \in \text{PROJECT}\; \exists\, w \in \text{WORKS\_ON} \; (w.\text{essn} = e.\text{ssn} \land w.\text{pno} = p.\text{pnumber}) \}
$$

**Why TRC is more declarative.** The user writes "what" (the conditions), not "how" (the order of operations). Both RA and TRC express the same queries (relational completeness) — every RA expression is expressible in safe TRC and vice versa.

---

## Worked example 5.3 — domain relational calculus

**Query (Q1) in DRC:**

$$
\{\, n \mid \exists\, s, a, d \;\; \text{EMPLOYEE}(s, n, a, d) \land \exists\, p, h \;\; \text{WORKS\_ON}(s, p, h) \land \exists\, pn, dn \;\; \text{PROJECT}(p, \text{'ProductX'}, dn) \,\}
$$

**Difference from TRC.** DRC quantifies over **domain values** rather than tuples; the same query looks similar but the variables are attribute values.

---

## 5.2 Quiz

1. Express "names of employees who work on projects in department 5" in relational algebra.
2. Express "departments with every employee earning > 30,000" in relational algebra.
3. Write a TRC formula for "employees who manage a department."
4. What is division in relational algebra used for? Give one example.
5. Why does relational algebra produce only **safe** expressions automatically, while calculus needs safety restrictions?

---

## Answer key

??? success "Click to reveal"

    1. $\pi_{\text{name}}(\text{EMPLOYEE} \bowtie \text{WORKS\_ON} \bowtie \sigma_{\text{dnum}=5}(\text{PROJECT}))$.
    2. $\pi_{\text{dno}}(\text{EMPLOYEE}) - \pi_{\text{dno}}(\sigma_{\text{salary} < 30000}(\text{EMPLOYEE}))$.
    3. $\{e.\text{name} \mid \text{EMPLOYEE}(e) \land \exists\, d \in \text{DEPARTMENT} \; (d.\text{mgr\_ssn} = e.\text{ssn})\}$.
    4. "For all" queries — e.g., employees who work on all projects.
    5. RA is built from operations that always produce finite relations from finite inputs; calculus allows expressions that range over infinite domains, so **safety** (domain independence) is needed.

---

## Further reading

- Elmasri & Navathe, §6.1–§6.6, §8.6 (calculus).
- Silberschatz et al., §2.6, §6.1.
- Ramakrishnan & Gehrke, §4.2–§4.3.