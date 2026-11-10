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
  js: assets/lecture-05.js

---

# ECEG-4191

## Chapter 5 — Relational Algebra and Calculus

---

## Learning outcomes

<div class="fragment">

1. Express queries using the **six fundamental relational-algebra operations**.

</div>

<div class="fragment">

2. Distinguish **theta, equi, natural**, and **outer** joins.

</div>

<div class="fragment">

3. Use **division** for "for all" queries.

</div>

<div class="fragment">

4. Translate between **relational algebra** and **tuple relational calculus**.

</div>

<div class="fragment">

5. Translate between **TRC** and **domain relational calculus**.

</div>

---

## 5.1 — Why Relational Algebra?

???+ note "Definition (Elmasri §6.1)"
    **Relational algebra** is a procedural query language: each query is a sequence of operations on relations, producing a new relation.

<div class="fragment">

???+ info "Why it matters"
    - It is the **theoretical foundation** of SQL.
    - It is the **intermediate language** for query optimization — every SQL query is translated to a relational-algebra tree.
    - It is **relationally complete** — any query expressible in relational calculus can be expressed in relational algebra.

</div>

<div class="fragment">

!!! tip "Procedural vs declarative"
    Relational algebra is **procedural** — you specify the operations. Relational calculus (and SQL) is **declarative** — you specify what you want. The two are equivalent in power.

</div>

---

## 5.2 — The Six Fundamental Operations

???+ note "The minimal complete set"
    | Operation | Symbol | Arity | Purpose |
    |---|---|---|---|
    | **Selection** | $\sigma$ | Unary | Filter tuples |
    | **Projection** | $\pi$ | Unary | Select columns |
    | **Union** | $\cup$ | Binary | Combine two relations |
    | **Difference** | $-$ | Binary | Tuples in first but not second |
    | **Cartesian product** | $\times$ | Binary | All pairings |
    | **Rename** | $\rho$ | Unary | Rename relations/attributes |

<div class="fragment">

!!! tip "Everything else is derived"
    Joins, intersection, division — all can be expressed using these six. Knowing which are *fundamental* is essential for understanding query optimization.

</div>

---

## 5.3 — Selection ($\sigma$)

???+ note "Definition"
    $\sigma_{\text{condition}}(R)$ returns the tuples of $R$ satisfying the boolean condition.

<div class="fragment">

???+ info "Conditions"
    - Comparison: $=, \ne, <, \le, >, \ge$.
    - Boolean: $\land, \lor, \lnot$.
    - Attribute names and constants.

</div>

<div class="fragment">

???+ info "Examples"
    - $\sigma_{\text{salary} > 40000}(\text{EMPLOYEE})$ — employees earning > 40k.
    - $\sigma_{\text{dno}=5 \land \text{sex}='F'}(\text{EMPLOYEE})$ — female employees in dept 5.

</div>

<div class="fragment">

!!! tip "Selection is commutative"
    $\sigma_{c_1}(\sigma_{c_2}(R)) = \sigma_{c_2}(\sigma_{c_1}(R))$. This is the first rule of query optimization.

</div>

---

## 5.4 — Projection ($\pi$)

???+ note "Definition"
    $\pi_{A_1, A_2, \ldots, A_k}(R)$ returns a relation with only the listed attributes. **Duplicates are removed** — the result is a set.

<div class="fragment">

???+ info "Examples"
    - $\pi_{\text{name, salary}}(\text{EMPLOYEE})$ — names and salaries.
    - $\pi_{\text{dno}}(\text{EMPLOYEE})$ — departments that have at least one employee (duplicates removed).

</div>

<div class="fragment">

!!! warning "Projection and duplicates"
    $\pi_{\text{dno}}(\text{EMPLOYEE})$ has **one tuple per department**, not per employee. This is because relational algebra is set-based. In SQL, you need `DISTINCT` because SQL's default is bag semantics.

</div>

---

## 5.5 — Union, Difference, Intersection

???+ note "Set-theoretic operations"
    For $R$ and $S$ with the **same schema** (union-compatible):
    
    - $R \cup S$ — tuples in $R$ or $S$ or both.
    - $R - S$ — tuples in $R$ but not $S$.
    - $R \cap S$ — tuples in both (derived: $R - (R - S)$).

<div class="fragment">

???+ info "Union-compatibility"
    Both relations must have the **same number of attributes** and **compatible domains** in each position. Attribute names may differ.

</div>

<div class="fragment">

???+ info "Example"
    Departments that have employees or projects:
    
    $$
    \pi_{\text{dno}}(\text{EMPLOYEE}) \cup \pi_{\text{dnum}}(\text{PROJECT}).
    $$

</div>

<div class="fragment">

!!! warning "Union removes duplicates"
    $R \cup S$ contains each tuple once. If you want to count duplicates, you need SQL's `UNION ALL` — not expressible in pure relational algebra.

</div>

---

## 5.6 — Cartesian Product ($\times$)

???+ note "Definition"
    $R \times S$ pairs every tuple of $R$ with every tuple of $S$. If $R$ has $n$ tuples and $S$ has $m$, $R \times S$ has $n \times m$ tuples.

<div class="fragment">

???+ info "Example"
    With 100 employees and 10 departments, `EMPLOYEE × DEPARTMENT` has 1,000 tuples — most of them meaningless (employees paired with unrelated departments).

</div>

<div class="fragment">

!!! tip "Cartesian product is the basis for joins"
    Every join is a Cartesian product followed by a selection. Query optimizers know this — and rarely materialize the full cross product.

</div>

<div class="fragment">

???+ warning "Size explosion"
    Cross products grow multiplicatively. On large tables, they are catastrophic. Always filter or join instead.

</div>

---

## 5.7 — Joins

???+ note "Four kinds of join"
    | Join | Symbol | Condition |
    |---|---|---|
    | **Theta join** | $R \bowtie_\theta S$ | Any predicate $\theta$ |
    | **Equijoin** | $R \bowtie_{A=B} S$ | Equality on attributes |
    | **Natural join** | $R \bowtie S$ | Equality on all **common** attributes, duplicates removed |
    | **Outer join** | $R ⟕ S$, etc. | Preserves unmatched tuples |

<div class="fragment">

???+ info "Example — theta join"
    $$
    \text{EMPLOYEE} \bowtie_{\text{salary} > 50000} \text{DEPARTMENT}
    $$
    
    Pairs every employee earning > 50k with every department (rarely useful).

</div>

<div class="fragment">

???+ info "Example — equijoin"
    $$
    \text{EMPLOYEE} \bowtie_{\text{dno} = \text{dnumber}} \text{DEPARTMENT}
    $$
    
    Correct pairing of employees with their departments.

</div>

<div class="fragment">

???+ info "Example — natural join"
    $$
    \text{EMPLOYEE} \bowtie \text{DEPARTMENT}
    $$
    
    Same as the equijoin above, but with duplicate `dno`/`dnumber` removed automatically.

</div>

---

## 5.8 — Outer Joins

???+ note "Why outer joins?"
    A natural join drops tuples that have no match on the other side. The **outer join** preserves them, padding the missing side with NULLs.

<div class="fragment">

???+ note "Three variants"
    | Variant | Symbol | Preserves |
    |---|---|---|
    | **Left outer** | $R ⟕ S$ | All of $R$ |
    | **Right outer** | $R ⟖ S$ | All of $S$ |
    | **Full outer** | $R ⟗ S$ | All of $R$ and $S$ |

</div>

<div class="fragment">

???+ info "Example"
    $$
    \text{EMPLOYEE} ⟕ \text{DEPARTMENT}
    $$
    
    Includes employees **with no department** (their `dno` is NULL in the padded DEPARTMENT columns). A regular natural join would drop them.

</div>

<div class="fragment">

!!! tip "Outer join in SQL"
    SQL supports all three variants: `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`. The default `INNER JOIN` is the natural join.

</div>

---

## 5.9 — Division ($\div$)

???+ note "Definition"
    For $R(A, B)$ and $S(B)$: $R \div S$ returns the **$A$-values** in $R$ that appear with **every** $B$-value in $S$.

<div class="fragment">

???+ info "Formal definition"
    $$
    R \div S = \{ t[A] \mid \forall s \in S,\ \exists r \in R : r[A] = t[A] \land r[B] = s[B] \}.
    $$

</div>

<div class="fragment">

???+ info "In relational algebra"
    $$
    R \div S = \pi_A(R) - \pi_A(\pi_A(R) \times S - R).
    $$
    
    "All $A$-values minus those that miss at least one $B$."

</div>

<div class="fragment">

!!! tip "Division = 'for all'"
    Division answers the "**for all**" questions: employees who work on **all** projects; students who took **every** required course; suppliers who supply **every** part.

</div>

---

## Worked example 5.1 — Division on COMPANY

!!! tip "Problem:"
    Find employees who work on **all** projects.

<div class="fragment">

???+ note "**Step 1 — schema.**"
    $\text{WORKS\_ON}(\text{essn}, \text{pno})$, $\text{PROJECT}(\text{pnumber})$.

</div>

<div class="fragment">

??? info "**Step 2 — set up division.**"
    $$
    \pi_{\text{essn}, \text{pno}}(\text{WORKS\_ON}) \div \pi_{\text{pnumber}}(\text{PROJECT}).
    $$

</div>

<div class="fragment">

??? info "**Step 3 — join with EMPLOYEE for names.**"
    $$
    \pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie (\pi_{\text{essn}, \text{pno}}(\text{WORKS\_ON}) \div \pi_{\text{pnumber}}(\text{PROJECT}))\bigr).
    $$

</div>

<div class="fragment">

??? success "**Interpretation.**"
    Employees whose `pno` set covers every `pnumber` in PROJECT. If a new project is added, only those who work on it are included — automatically.

</div>

---

## 5.10 — Additional Operations

???+ note "Derived operations"
    | Operation | Symbol | Definition |
    |---|---|---|
    | **Intersection** | $\cap$ | $R \cap S = R - (R - S)$ |
    | **Natural join** | $\bowtie$ | $\pi_{\text{all}} (\sigma_{\text{equal common attrs}}(R \times S))$ |
    | **Assignment** | $:=$ | Name an intermediate result |
    | **Aggregate** | $\mathcal{F}$ | Count, sum, avg, min, max |

<div class="fragment">

???+ info "Aggregate operator"
    $\mathcal{G}_{\text{grouping attrs}} \mathcal{F}_{\text{agg}}(\text{relation})$:
    
    $\mathcal{G}_{\text{dno}} \mathcal{F}_{\text{COUNT(ssn), AVG(salary)}}(\text{EMPLOYEE})$
    
    Groups by department; counts employees; averages salaries.

</div>

<div class="fragment">

!!! tip "Relational algebra is closed"
    Every operation takes relations and returns relations — so operations compose freely.

</div>

---

## Worked example 5.2 — Five COMPANY queries

!!! tip "Company schema:"
    `EMPLOYEE(ssn, name, salary, dno)`, `DEPARTMENT(dnumber, dname, mgr_ssn)`, `PROJECT(pnumber, pname, dnum)`, `WORKS_ON(essn, pno, hours)`.

<div class="fragment">

???+ note "**Q1 — names of employees in 'Research'**"
    $$
    \pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie \sigma_{\text{dname}=\text{'Research'}}(\text{DEPARTMENT})\bigr)
    $$

</div>

<div class="fragment">

??? info "**Q2 — names of employees who work on 'ProductX'**"
    $$
    \pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie \text{WORKS\_ON} \bowtie \sigma_{\text{pname}=\text{'ProductX'}}(\text{PROJECT})\bigr)
    $$

</div>

<div class="fragment">

??? info "**Q3 — employees who do not work on any project**"
    $$
    \pi_{\text{name}}(\text{EMPLOYEE}) - \pi_{\text{name}}(\text{EMPLOYEE} \bowtie \text{WORKS\_ON})
    $$

</div>

<div class="fragment">

??? info "**Q4 — department with the highest-paid manager**"
    Let $S := \mathcal{G}_{\text{dno}} \mathcal{F}_{\text{MAX(salary)}}(\text{EMPLOYEE})$.
    $$
    \pi_{\text{dname}}\Bigl(\sigma_{\text{salary} = \text{max}}(\text{EMPLOYEE} \bowtie S) \bowtie \text{DEPARTMENT}\Bigr)
    $$

</div>

<div class="fragment">

??? info "**Q5 — employees who work on all projects**"
    $$
    \pi_{\text{name}}\bigl(\text{EMPLOYEE} \bowtie (\pi_{\text{essn}, \text{pno}}(\text{WORKS\_ON}) \div \pi_{\text{pnumber}}(\text{PROJECT}))\bigr)
    $$

</div>

---

## 5.11 — Tuple Relational Calculus

???+ note "Definition (Elmasri §6.6)"
    **TRC** is a declarative language: a query is a set of tuples satisfying a predicate.
    
    $$
    \{ t \mid P(t) \}
    $$
    
    where $t$ is a tuple variable.

<div class="fragment">

???+ info "Quantifiers"
    - $\exists t \in R \; P(t)$ — there exists a tuple in $R$ satisfying $P$.
    - $\forall t \in R \; P(t)$ — every tuple in $R$ satisfies $P$.

</div>

<div class="fragment">

???+ info "Example"
    Employees earning > 40k:
    $$
    \{ t \mid t \in \text{EMPLOYEE} \land t[\text{salary}] > 40000 \}.
    $$

</div>

<div class="fragment">

!!! tip "TRC is declarative"
    You specify **what** — not how. The DBMS decides the execution order. This is the same paradigm as SQL.

</div>

---

## Worked example 5.3 — TRC queries

!!! tip "Express in TRC:"

<div class="fragment">

???+ note "**Q1 — names of employees in 'Research'**"
    $$
    \{ t[\text{name}] \mid t \in \text{EMPLOYEE} \land \exists d \in \text{DEPARTMENT} \; (d[\text{dnumber}] = t[\text{dno}] \land d[\text{dname}] = \text{'Research'}) \}
    $$

</div>

<div class="fragment">

??? info "**Q2 — employees who work on all projects**"
    $$
    \{ t[\text{name}] \mid t \in \text{EMPLOYEE} \land \forall p \in \text{PROJECT} \; \exists w \in \text{WORKS\_ON} \; (w[\text{essn}] = t[\text{ssn}] \land w[\text{pno}] = p[\text{pnumber}]) \}
    $$

</div>

<div class="fragment">

??? info "**Q3 — departments with no employees**"
    $$
    \{ d[\text{dname}] \mid d \in \text{DEPARTMENT} \land \lnot \exists e \in \text{EMPLOYEE} \; (e[\text{dno}] = d[\text{dnumber}]) \}
    $$

</div>

<div class="fragment">

??? success "**Notice the pattern.**"
    - "There exists" → $\exists$.
    - "For all" → $\forall$.
    - "Does not exist" → $\lnot \exists$.

</div>

---

## 5.12 — Domain Relational Calculus

???+ note "Definition"
    **DRC** quantifies over **domain values**, not tuples.
    
    $$
    \{ \langle x_1, x_2, \ldots, x_n \rangle \mid P(x_1, \ldots, x_n) \}
    $$

<div class="fragment">

???+ info "Example"
    Employees earning > 40k:
    $$
    \{ \langle n \rangle \mid \exists s, a, d \; (\text{EMPLOYEE}(s, n, a, d) \land a > 40000) \}
    $$
    
    Note: `EMPLOYEE(s, n, a, d)` is a predicate asserting "there is a tuple with these values".

</div>

<div class="fragment">

!!! tip "TRC vs DRC"
    Both are **relationally complete** — they express the same set of queries. TRC is easier for large schemas (tuple variables); DRC is easier for formal proofs (pure first-order logic).

</div>

<div class="fragment">

???+ warning "Safety"
    DRC expressions must be **safe** — every variable must be drawn from a finite domain. Without safety, expressions can range over infinite domains and produce infinite results.

</div>

---

## 5.13 — Equivalence of Languages

???+ note "Three equivalent languages"
    | Language | Style | Power |
    |---|---|---|
    | **Relational algebra** | Procedural | Same |
    | **Tuple relational calculus** | Declarative | Same |
    | **Domain relational calculus** | Declarative | Same |

<div class="fragment">

???+ abstract "Codd's theorem"
    Relational algebra, TRC, and DRC are **relationally complete** — they express exactly the same set of queries (those expressible in first-order logic over finite relations).

</div>

<div class="fragment">

!!! tip "Why this matters"
    - **Query optimization**: converting SQL (calculus) to a plan (algebra).
    - **Correctness proofs**: showing two queries are equivalent.
    - **Language design**: SQL is essentially a syntactically sugared TRC.

</div>

---

## Chapter 5 — key takeaways

<div class="fragment">

**1.** Six **fundamental operations**: $\sigma, \pi, \cup, -, \times, \rho$. Everything else is derived.

</div>

<div class="fragment">

**2.** **Selection** filters tuples; **projection** selects columns and **removes duplicates**.

</div>

<div class="fragment">

**3.** **Joins**: theta (any predicate), equi (equality), natural (equality on common attrs), outer (preserves unmatched).

</div>

<div class="fragment">

**4.** **Division** answers "for all" questions — employees who work on all projects.

</div>

<div class="fragment">

**5.** **TRC** quantifies over tuples; **DRC** quantifies over values. Both are declarative.

</div>

<div class="fragment">

**6.** **Codd's theorem**: relational algebra, TRC, and DRC are **relationally complete** — same expressive power.

</div>

---

## Next — Chapter 6

**SQL.**

<div class="fragment">

- **DDL**: `CREATE`, `ALTER`, `DROP`; constraints.
- **DML**: `SELECT` with `WHERE`, joins, aggregates, `GROUP BY`, `HAVING`, subqueries, `EXISTS`, set operations, `NULL` handling.
- **Data modification**: `INSERT`, `UPDATE`, `DELETE`.

</div>

<div class="fragment">

Chapter 5 gave the **theory**. Chapter 6 gives the **language** every database engineer uses every day.

</div>