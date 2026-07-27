# MODULE 4 — MySQL: SQL Fundamentals, Constraints, DDL/DML

## 1. Database & RDBMS Basics
- **Database**: organized collection of data.
- **RDBMS** (Relational DBMS): stores data in **tables** (rows × columns), enforces relationships via keys. Examples: MySQL, Oracle, PostgreSQL.
- **SQL** (Structured Query Language): language to talk to RDBMS. It is **declarative** — you say *what* you want, not *how* to get it.

## 2. SQL Command Categories — THE #1 asked concept

| Category | Full Form | Commands | Purpose |
|---|---|---|---|
| **DDL** | Data Definition Language | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` | Defines/modifies structure (schema) |
| **DML** | Data Manipulation Language | `INSERT`, `UPDATE`, `DELETE` | Manipulates data inside tables |
| **DQL** | Data Query Language | `SELECT` | Retrieves data (sometimes clubbed under DML) |
| **DCL** | Data Control Language | `GRANT`, `REVOKE` | Controls access/permissions |
| **TCL** | Transaction Control Language | `COMMIT`, `ROLLBACK`, `SAVEPOINT` | Manages transactions |

**Capgemini trap:** `TRUNCATE` is **DDL**, not DML — even though it "deletes data," it resets the table structure and can't be rolled back (in most engines) because it's a structural operation, not a row-by-row DML operation.

**Trap #2:** `DROP` removes the table + structure + data permanently. `TRUNCATE` removes all rows but **keeps structure**. `DELETE` removes rows (optionally filtered) and **can be rolled back** (it's DML, logged row-by-row).

| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Rows (with WHERE optional) | All rows | Table + structure |
| WHERE clause | Yes | No | No |
| Rollback | Yes (logged) | No (in most engines) | No |
| Resets AUTO_INCREMENT | No | Yes | N/A (table gone) |

## 3. Constraints — very frequently asked

| Constraint | Meaning | Example |
|---|---|---|
| **PRIMARY KEY** | Uniquely identifies each row. Combines `UNIQUE` + `NOT NULL`. Only **one** per table (can be composite). | `id INT PRIMARY KEY` |
| **FOREIGN KEY** | Links a column to a PRIMARY KEY in another table. Enforces referential integrity. | `dept_id INT REFERENCES dept(id)` |
| **UNIQUE** | No duplicate values allowed, but **NULL is allowed** (unlike primary key). | `email VARCHAR(50) UNIQUE` |
| **NOT NULL** | Column cannot store NULL. | `name VARCHAR(50) NOT NULL` |
| **CHECK** | Validates a condition before insert/update. | `age INT CHECK (age >= 18)` |
| **DEFAULT** | Sets a default value if none supplied. | `status VARCHAR(10) DEFAULT 'ACTIVE'` |

**Trap:** A table can have multiple `UNIQUE` columns, but only **one** `PRIMARY KEY` (composite keys count as one). A `UNIQUE` column **can** hold one NULL value (or multiple NULLs in MySQL specifically — MySQL treats NULLs as distinct for UNIQUE), a `PRIMARY KEY` column can **never** hold NULL.

## 4. Core DDL Syntax
```sql
CREATE TABLE employee (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    salary DECIMAL(10,2) DEFAULT 0,
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES department(id)
);

ALTER TABLE employee ADD COLUMN email VARCHAR(100);
ALTER TABLE employee MODIFY COLUMN salary DECIMAL(12,2);
ALTER TABLE employee DROP COLUMN email;

DROP TABLE employee;
TRUNCATE TABLE employee;
```

## 5. Core DML/DQL Syntax
```sql
INSERT INTO employee (id, name, salary) VALUES (1, 'Asha', 50000);
UPDATE employee SET salary = 55000 WHERE id = 1;
DELETE FROM employee WHERE id = 1;

SELECT name, salary FROM employee
WHERE salary > 40000
ORDER BY salary DESC
LIMIT 5;
```
- **WHERE** filters rows **before** grouping.
- **ORDER BY** default is ascending (`ASC`); use `DESC` for descending.
- **LIMIT** restricts number of returned rows (MySQL-specific; Oracle uses `ROWNUM`/`FETCH`).

## 6. GROUP BY, HAVING, DISTINCT — classic trap zone
```sql
SELECT dept_id, COUNT(*) AS emp_count
FROM employee
GROUP BY dept_id
HAVING COUNT(*) > 3;
```
- **WHERE** filters rows before grouping; **HAVING** filters **groups after** aggregation.
- **Trap:** You **cannot** use an aggregate function (`COUNT`, `SUM`) directly inside `WHERE`. Must use `HAVING`.
- **DISTINCT** removes duplicate rows from result set: `SELECT DISTINCT dept_id FROM employee;`

## 7. Common Functions

| Type | Examples |
|---|---|
| Aggregate | `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` |
| String | `CONCAT()`, `UPPER()`, `LOWER()`, `LENGTH()`, `SUBSTRING()`, `TRIM()` |
| Date | `NOW()`, `CURDATE()`, `DATEDIFF()`, `DATE_ADD()` |

**Trap:** `COUNT(*)` counts all rows including NULLs; `COUNT(column_name)` skips NULLs in that column.

## 8. LIKE & Wildcards
- `%` = zero or more characters. `_` = exactly one character.
- `WHERE name LIKE 'A%'` → names starting with A.
- `WHERE name LIKE '_a%'` → second letter is 'a'.

## One-Page Revision (DDL/DML/Constraints)
- DDL = structure (CREATE, ALTER, DROP, TRUNCATE). DML = data (INSERT, UPDATE, DELETE). DCL = permissions (GRANT, REVOKE). TCL = transactions (COMMIT, ROLLBACK).
- TRUNCATE is DDL (trap!), resets identity, no WHERE, generally not rollback-able.
- PRIMARY KEY = UNIQUE + NOT NULL, only one per table. FOREIGN KEY links to another table's PK. UNIQUE allows NULL, PK doesn't.
- WHERE filters rows before grouping; HAVING filters groups after aggregation; can't use aggregate functions in WHERE.
- COUNT(*) counts NULLs too; COUNT(col) does not.
- DELETE = DML, filterable, rollback-able. TRUNCATE/DROP = DDL, not filterable, generally not rollback-able.

---

# MCQs — SQL Fundamentals, Constraints, DDL/DML (20 Questions)

**Q1.** [Easy | DDL vs DML] Which category does `TRUNCATE` belong to?
A) DML B) DDL C) DCL D) TCL
**Answer: B**
*Explanation:* TRUNCATE removes all rows and resets table structure/auto-increment; it is treated as a DDL operation, not logged row-by-row like DML.
*Why others wrong:* A) DML operations (INSERT/UPDATE/DELETE) are row-level and loggable. C) DCL handles permissions (GRANT/REVOKE). D) TCL handles transactions (COMMIT/ROLLBACK).

**Q2.** [Easy | Constraints] Which constraint allows only one NULL-free, unique column per table?
A) UNIQUE B) CHECK C) PRIMARY KEY D) DEFAULT
**Answer: C**
*Explanation:* PRIMARY KEY = UNIQUE + NOT NULL, and only one is allowed per table (composite keys count as one).
*Why others wrong:* A) A table can have multiple UNIQUE columns and they allow NULL. B) CHECK validates a condition, not uniqueness. D) DEFAULT just supplies a fallback value.

**Q3.** [Easy | DML] Which statement removes rows and CAN be rolled back?
A) TRUNCATE B) DROP C) DELETE D) ALTER
**Answer: C**
*Explanation:* DELETE is DML and logs each row removal, making it rollback-able within a transaction.
*Why others wrong:* A, B) DDL operations, generally auto-committed and not rollback-able. D) ALTER changes structure, not row data.

**Q4.** [Medium | WHERE vs HAVING] Which clause filters groups AFTER aggregation?
A) WHERE B) GROUP BY C) HAVING D) ORDER BY
**Answer: C**
*Explanation:* HAVING is applied after GROUP BY has formed groups and aggregate functions have been computed.
*Why others wrong:* A) WHERE filters individual rows before grouping. B) GROUP BY forms the groups, doesn't filter them. D) ORDER BY only sorts the final result.

**Q5.** [Medium | Trap] Which query is INVALID?
A) `SELECT dept_id, COUNT(*) FROM emp GROUP BY dept_id HAVING COUNT(*) > 2;`
B) `SELECT * FROM emp WHERE COUNT(*) > 2;`
C) `SELECT dept_id FROM emp GROUP BY dept_id;`
D) `SELECT DISTINCT dept_id FROM emp;`
**Answer: B**
*Explanation:* Aggregate functions like COUNT() cannot be used directly in WHERE; they require HAVING after grouping.
*Why others wrong:* A, C, D are all syntactically valid SQL patterns.

**Q6.** [Medium | Keys] A FOREIGN KEY column value must:
A) Always be unique in its own table B) Match a value existing in the referenced table's key column (or be NULL) C) Never be NULL D) Auto-increment
**Answer: B**
*Explanation:* Foreign keys enforce referential integrity — the value must exist in the referenced table's primary/unique key, or be NULL if the FK column allows it.
*Why others wrong:* A) FK values commonly repeat (many rows can reference the same parent). C) NULL is allowed unless explicitly restricted. D) Auto-increment is unrelated to FK behavior.

**Q7.** [Medium | Functions] What does `COUNT(salary)` do differently from `COUNT(*)` if some salary values are NULL?
A) No difference B) COUNT(salary) excludes NULL rows, COUNT(*) includes them C) COUNT(*) throws an error D) COUNT(salary) counts only distinct values
**Answer: B**
*Explanation:* COUNT(column) ignores NULLs in that column, while COUNT(*) counts all rows regardless of NULL values.
*Why others wrong:* A) They do differ when NULLs exist. C) COUNT(*) never errors on NULLs. D) That would be COUNT(DISTINCT salary).

**Q8.** [Medium | LIKE] `WHERE name LIKE '_a%'` matches:
A) Names containing 'a' anywhere B) Names where the second character is 'a' C) Names starting with 'a' D) Names ending with 'a'
**Answer: B**
*Explanation:* `_` matches exactly one character (the first), and `a` must be the second character, followed by anything (`%`).
*Why others wrong:* A) That would need `%a%`. C) That would need `a%`. D) That would need `%a`.

**Q9.** [Medium | Trap] Which statement about UNIQUE vs PRIMARY KEY is TRUE?
A) Both disallow NULL entirely B) UNIQUE allows NULL, PRIMARY KEY does not C) PRIMARY KEY allows NULL, UNIQUE does not D) Neither can be applied to more than one column
**Answer: B**
*Explanation:* UNIQUE permits NULL values (MySQL treats multiple NULLs as distinct), while PRIMARY KEY strictly disallows NULL.
*Why others wrong:* A, C are factually reversed/incorrect. D) Both can be composite (multi-column).

**Q10.** [Hard | Scenario] You run `TRUNCATE TABLE orders;` inside a transaction and then `ROLLBACK;`. What happens in most MySQL setups (InnoDB)?
A) Data is restored B) Data remains truncated — TRUNCATE typically cannot be rolled back C) Only half the rows are restored D) An error is thrown
**Answer: B**
*Explanation:* TRUNCATE is a DDL statement that implicitly commits and, in most engines, cannot be undone by ROLLBACK.
*Why others wrong:* A, C) Suggest partial/rollback-ability that doesn't apply to TRUNCATE. D) No error occurs; the operation just isn't reversible.

**Q11.** [Easy | DCL] `GRANT` and `REVOKE` belong to which category?
A) DDL B) DML C) DCL D) TCL
**Answer: C**
*Explanation:* DCL (Data Control Language) manages access permissions via GRANT and REVOKE.
*Why others wrong:* A) DDL defines structure. B) DML manipulates data. D) TCL manages transactions.

**Q12.** [Easy | TCL] Which command permanently saves a transaction's changes?
A) ROLLBACK B) COMMIT C) SAVEPOINT D) GRANT
**Answer: B**
*Explanation:* COMMIT makes all changes in the current transaction permanent.
*Why others wrong:* A) ROLLBACK undoes changes. C) SAVEPOINT marks a point to roll back to, doesn't finalize. D) GRANT is unrelated to transactions.

**Q13.** [Medium | ALTER] Which command changes an existing column's data type?
A) `ALTER TABLE t ADD COLUMN` B) `ALTER TABLE t MODIFY COLUMN` C) `ALTER TABLE t DROP COLUMN` D) `UPDATE t SET COLUMN`
**Answer: B**
*Explanation:* MODIFY COLUMN (MySQL syntax) changes an existing column's definition, including data type.
*Why others wrong:* A) Adds a new column. C) Removes a column. D) UPDATE changes data values, not column type.

**Q14.** [Medium | Scenario] A column is defined `email VARCHAR(50) UNIQUE`. You insert two rows with `email = NULL`. What happens?
A) Error: duplicate value B) Both inserts succeed — MySQL treats NULLs as distinct for UNIQUE C) Only first insert succeeds D) Column is auto-converted to allow duplicates
**Answer: B**
*Explanation:* MySQL's UNIQUE constraint does not consider NULL values as duplicates of each other, so multiple NULLs are allowed.
*Why others wrong:* A, C) Would be true for non-NULL duplicate values, not NULL. D) No such auto-conversion occurs.

**Q15.** [Medium | ORDER BY] Default sort order of `ORDER BY salary` (no keyword) is:
A) Descending B) Random C) Ascending D) By insertion order
**Answer: C**
*Explanation:* ORDER BY defaults to ASC (ascending) when no direction keyword is specified.
*Why others wrong:* A) Requires explicit DESC. B, D) SQL does not guarantee these behaviors for ORDER BY.

**Q16.** [Hard | Trap] Which of these can appear in a `SELECT` list WITHOUT being in `GROUP BY` (standard SQL / strict mode)?
A) A non-aggregated column not in GROUP BY B) An aggregate function like `SUM(col)` C) Any arbitrary column D) A column from a different table not joined
**Answer: B**
*Explanation:* Aggregate functions compute one value per group and are always valid in SELECT alongside GROUP BY, regardless of grouping columns.
*Why others wrong:* A) In strict SQL mode, non-aggregated, non-grouped columns cause an error (MySQL's ONLY_FULL_GROUP_BY). C) Same issue as A. D) Referencing an unjoined table's column is invalid regardless of GROUP BY.

**Q17.** [Medium | AUTO_INCREMENT] After `TRUNCATE TABLE employee;`, what happens to the AUTO_INCREMENT counter (typical MySQL/InnoDB behavior)?
A) Stays the same as before truncation B) Resets to its starting value (usually 1) C) Becomes NULL D) Doubles
**Answer: B**
*Explanation:* TRUNCATE resets the AUTO_INCREMENT counter because it deallocates and recreates the table's storage, unlike DELETE.
*Why others wrong:* A) That's DELETE's behavior, not TRUNCATE's. C, D) Not real MySQL behaviors.

**Q18.** [Medium | Scenario] Which single query correctly finds departments with more than 5 employees?
A) `SELECT dept_id FROM emp WHERE COUNT(*) > 5;`
B) `SELECT dept_id, COUNT(*) FROM emp GROUP BY dept_id HAVING COUNT(*) > 5;`
C) `SELECT dept_id FROM emp ORDER BY COUNT(*) > 5;`
D) `SELECT dept_id FROM emp GROUP BY dept_id WHERE COUNT(*) > 5;`
**Answer: B**
*Explanation:* Correct SQL clause order is SELECT → GROUP BY → HAVING, and HAVING is required for filtering on aggregates.
*Why others wrong:* A, D) Misuse WHERE for aggregate filtering / wrong clause order. C) ORDER BY cannot filter results.

**Q19.** [Easy | Terminology] What does DDL stand for?
A) Data Definition Language B) Data Deletion Logic C) Database Design Layer D) Data Duplication Language
**Answer: A**
*Explanation:* DDL (Data Definition Language) defines and modifies database structure/schema.
*Why others wrong:* B, C, D are fabricated, non-standard expansions.

**Q20.** [Hard | Integration] Order these from broadest structural impact to least: (1) DELETE FROM t; (2) DROP TABLE t; (3) TRUNCATE TABLE t; (4) UPDATE t SET x=1;
A) 2 → 3 → 1 → 4 B) 3 → 2 → 1 → 4 C) 2 → 1 → 3 → 4 D) 4 → 1 → 3 → 2
**Answer: A**
*Explanation:* DROP removes the whole table (most impact) → TRUNCATE clears all rows but keeps structure → DELETE removes selected/all rows (row-level DML) → UPDATE only modifies values in place (least structural impact).
*Why others wrong:* B, C, D place operations out of the correct impact-severity order.

---

# MODULE 4 — MySQL: Joins, Subqueries, Views, Indexes, Normalization, ACID

## 1. Joins — THE most frequently asked SQL topic

| Join | Returns |
|---|---|
| **INNER JOIN** | Only matching rows in **both** tables |
| **LEFT JOIN** | All rows from left table + matched rows from right (unmatched → NULL) |
| **RIGHT JOIN** | All rows from right table + matched rows from left (unmatched → NULL) |
| **FULL JOIN** | All rows from both tables (matched + unmatched from both). *(MySQL has no native FULL JOIN — simulate with `UNION` of LEFT and RIGHT JOIN)* |
| **SELF JOIN** | Table joined with itself (using aliases) |
| **CROSS JOIN** | Cartesian product — every row of A with every row of B |

```sql
SELECT e.name, d.dept_name
FROM employee e
INNER JOIN department d ON e.dept_id = d.id;

SELECT e.name, d.dept_name
FROM employee e
LEFT JOIN department d ON e.dept_id = d.id;
```

**Capgemini trap:** MySQL does **not** support `FULL OUTER JOIN` directly. The workaround is:
```sql
SELECT * FROM a LEFT JOIN b ON a.id=b.id
UNION
SELECT * FROM a RIGHT JOIN b ON a.id=b.id;
```

**Trap #2:** In a LEFT JOIN, if there's no match in the right table, the right table's columns return **NULL** — the row from the left table is still kept.

## 2. Subqueries
A query nested inside another query.
```sql
-- Single-row subquery
SELECT name FROM employee
WHERE salary > (SELECT AVG(salary) FROM employee);

-- Multi-row subquery
SELECT name FROM employee
WHERE dept_id IN (SELECT id FROM department WHERE location = 'Bangalore');

-- Correlated subquery (references outer query)
SELECT e1.name FROM employee e1
WHERE salary > (SELECT AVG(salary) FROM employee e2 WHERE e2.dept_id = e1.dept_id);
```
- **Correlated subquery**: re-executes once **per outer row** — slower but powerful for row-context comparisons.
- **Trap:** `=` works only with subqueries returning a single value; use `IN` for multi-row results, else you get a runtime error ("subquery returns more than 1 row").

## 3. Views
A **virtual table** based on a stored SQL query. Doesn't store data itself (usually).
```sql
CREATE VIEW high_earners AS
SELECT name, salary FROM employee WHERE salary > 50000;

SELECT * FROM high_earners;
```
- Simplifies complex queries, adds a security layer (restrict column/row access), doesn't duplicate data.
- **Trap:** Views are generally **updatable only** if based on a single table without aggregates/GROUP BY/DISTINCT/joins in some cases — updating a complex view can fail or be restricted.

## 4. Indexes
Speed up data **retrieval** at the cost of slower **writes** (INSERT/UPDATE/DELETE) and extra storage.
```sql
CREATE INDEX idx_salary ON employee(salary);
```
- Primary Key automatically creates a **clustered index** (in InnoDB, data physically ordered by PK).
- Indexes work like a book's index — avoid full table scans.
- **Trap:** Over-indexing hurts write performance because every INSERT/UPDATE must also update all relevant indexes.

## 5. Normalization — very frequently asked

| Normal Form | Rule |
|---|---|
| **1NF** | Atomic values only; no repeating groups/multi-valued columns |
| **2NF** | 1NF + no partial dependency (non-key attributes depend on the **whole** composite key) |
| **3NF** | 2NF + no transitive dependency (non-key attributes depend **only** on the key, not on other non-key attributes) |
| **BCNF** | Stricter 3NF — every determinant is a candidate key |

**Goal:** Eliminate data redundancy and update anomalies by splitting tables logically.

**Trap:** "Normalization always improves performance" → **False**. Normalization reduces redundancy but can require more JOINs, which may reduce read performance — that's why **denormalization** is sometimes deliberately used for read-heavy systems.

## 6. Transactions & ACID — very frequently asked

| Property | Meaning |
|---|---|
| **Atomicity** | Transaction is all-or-nothing — either all statements succeed or none do |
| **Consistency** | DB moves from one valid state to another; constraints/rules always hold |
| **Isolation** | Concurrent transactions don't interfere with each other's intermediate state |
| **Durability** | Once committed, changes survive even a crash (persisted to disk) |

```sql
START TRANSACTION;
UPDATE account SET balance = balance - 500 WHERE id = 1;
UPDATE account SET balance = balance + 500 WHERE id = 2;
COMMIT;   -- or ROLLBACK; on failure
```
**Trap:** Money-transfer is the classic ACID example — **Atomicity** ensures if the second UPDATE fails, the first is rolled back too (no money "disappears").

## One-Page Revision (Joins/Subqueries/Views/Indexes/Normalization/ACID)
- INNER = only matches. LEFT = all left + matched right (NULL if none). RIGHT = mirror of LEFT. FULL = both sides, not natively in MySQL (simulate via UNION).
- Subquery with `=` needs single value; use `IN` for multiple rows. Correlated subquery runs per outer row.
- Views = virtual tables from a stored query; simplify + secure, don't duplicate data; complex views may not be updatable.
- Indexes speed up reads, slow down writes, cost storage; PK = clustered index in InnoDB.
- Normalization: 1NF (atomic) → 2NF (no partial dependency) → 3NF (no transitive dependency) → BCNF. Reduces redundancy, may increase JOINs.
- ACID = Atomicity, Consistency, Isolation, Durability — guarantees reliable transactions.

---

# MCQs — Joins, Subqueries, Views, Indexes, Normalization, ACID (25 Questions)

**Q1.** [Easy | Joins] Which join returns only rows with matches in BOTH tables?
A) LEFT JOIN B) RIGHT JOIN C) INNER JOIN D) CROSS JOIN
**Answer: C**
*Explanation:* INNER JOIN returns only the rows where the join condition is satisfied in both tables.
*Why others wrong:* A, B) Include all rows from one side regardless of match. D) Returns Cartesian product, no matching condition needed.

**Q2.** [Easy | Joins] In a LEFT JOIN, unmatched rows from the right table show as:
A) 0 B) Empty string C) NULL D) Error
**Answer: C**
*Explanation:* When there's no matching row in the right table, its columns are filled with NULL, while the left row is still returned.
*Why others wrong:* A, B) SQL uses NULL to represent "no value," not 0 or empty string. D) No error occurs; this is expected LEFT JOIN behavior.

**Q3.** [Medium | Trap] Does MySQL support `FULL OUTER JOIN` natively?
A) Yes, with the `FULL JOIN` keyword B) No — must simulate using UNION of LEFT and RIGHT JOIN C) Yes, but only in stored procedures D) No, MySQL has no way to achieve this
**Answer: B**
*Explanation:* MySQL lacks native FULL OUTER JOIN support; the standard workaround is UNIONing a LEFT JOIN and RIGHT JOIN result.
*Why others wrong:* A, C) MySQL has no such native keyword/context. D) It IS achievable via the UNION workaround.

**Q4.** [Medium | Subquery] Which keyword should you use when a subquery may return multiple rows?
A) `=` B) `IN` C) `LIKE` D) `IS`
**Answer: B**
*Explanation:* IN correctly handles comparison against a list of values, which is what a multi-row subquery returns.
*Why others wrong:* A) `=` errors out if the subquery returns more than one row. C, D) Not designed for multi-value subquery comparisons.

**Q5.** [Medium | Subquery] A correlated subquery is one that:
A) Runs once before the outer query B) References columns from the outer query and re-executes per outer row C) Cannot use WHERE D) Always returns multiple rows
**Answer: B**
*Explanation:* A correlated subquery depends on the outer query's current row value, so it logically re-runs for each row processed by the outer query.
*Why others wrong:* A) That describes a non-correlated (independent) subquery. C) WHERE is commonly used inside correlated subqueries. D) It can return single or multiple values depending on context.

**Q6.** [Easy | Views] A VIEW is best described as:
A) A physical copy of table data B) A virtual table based on a stored query C) An index type D) A stored procedure
**Answer: B**
*Explanation:* Views don't store data themselves (in the standard case); they present the result of a stored SELECT query as if it were a table.
*Why others wrong:* A) That would be a materialized view or a real table copy, not a standard view. C) Indexes are unrelated data structures. D) Stored procedures are executable code blocks, not queries presented as tables.

**Q7.** [Medium | Views] Which is a valid reason to use a VIEW?
A) Increase raw storage usage intentionally B) Simplify complex joins and restrict column/row visibility C) Physically duplicate data for backup D) Replace the need for indexes
**Answer: B**
*Explanation:* Views abstract complex queries into a simple interface and can restrict which columns/rows a user sees, adding a security layer.
*Why others wrong:* A) Views don't intentionally increase storage. C) Views are not a backup mechanism. D) Views don't replace indexing for performance.

**Q8.** [Medium | Index] What is the main tradeoff of adding an index?
A) Faster reads, slower writes B) Slower reads, faster writes C) No effect on performance D) Reduces storage usage
**Answer: A**
*Explanation:* Indexes speed up SELECT queries by avoiding full table scans, but every INSERT/UPDATE/DELETE must also maintain the index, slowing writes, and indexes consume extra storage.
*Why others wrong:* B) Reverses the actual tradeoff. C) Indexes have a clear, measurable performance effect. D) Indexes increase, not reduce, storage usage.

**Q9.** [Medium | Index] In InnoDB, the PRIMARY KEY automatically creates:
A) A hash index only B) A clustered index (data physically ordered by PK) C) No index at all D) A view
**Answer: B**
*Explanation:* InnoDB stores table data physically ordered by the primary key, functioning as a clustered index.
*Why others wrong:* A) InnoDB primarily uses B-tree, not hash, for this. C) A PK always creates an index automatically. D) Unrelated concept.

**Q10.** [Easy | Normalization] 1NF requires:
A) No partial dependency B) Atomic column values, no repeating groups C) No transitive dependency D) Every determinant is a candidate key
**Answer: B**
*Explanation:* First Normal Form requires that each column hold indivisible (atomic) values and that there be no repeating groups/multi-valued columns.
*Why others wrong:* A) That's a 2NF requirement. C) That's a 3NF requirement. D) That describes BCNF.

**Q11.** [Medium | Normalization] 2NF adds which requirement on top of 1NF?
A) No transitive dependency B) No partial dependency on a composite key C) Atomic values D) Every determinant must be a candidate key
**Answer: B**
*Explanation:* 2NF requires that non-key attributes depend on the entire composite primary key, not just part of it (no partial dependency).
*Why others wrong:* A) That's 3NF. C) That's 1NF, already assumed. D) That's BCNF.

**Q12.** [Medium | Trap] "Normalization always improves query performance." True or False?
A) True B) False
**Answer: B**
*Explanation:* Normalization reduces redundancy but can increase the number of JOINs needed for queries, sometimes hurting read performance — which is why denormalization is used in read-heavy systems.
*Why others wrong:* A is the trap; Capgemini likes testing that normalization's benefit is data integrity/redundancy reduction, not guaranteed speed.

**Q13.** [Easy | ACID] Which ACID property ensures a transaction is "all or nothing"?
A) Consistency B) Isolation C) Atomicity D) Durability
**Answer: C**
*Explanation:* Atomicity guarantees that either every statement in a transaction succeeds, or none of them take effect.
*Why others wrong:* A) Consistency ensures valid state transitions. B) Isolation concerns concurrent transaction visibility. D) Durability concerns persistence after commit.

**Q14.** [Medium | ACID] Which property ensures committed data survives a system crash?
A) Atomicity B) Consistency C) Isolation D) Durability
**Answer: D**
*Explanation:* Durability guarantees that once a transaction is committed, its changes are permanently saved even if the system crashes immediately after.
*Why others wrong:* A) Concerns all-or-nothing execution, not crash survival. B) Concerns valid state, not persistence. C) Concerns concurrency, not persistence.

**Q15.** [Medium | Scenario] A bank transfer debits account A and credits account B in one transaction. The credit step fails. What should happen under ACID?
A) Only the debit is kept B) Both operations are rolled back (Atomicity) C) The system retries indefinitely D) Nothing, both were already committed separately
**Answer: B**
*Explanation:* Atomicity ensures that if any part of the transaction fails, the entire transaction is rolled back, preventing money from "disappearing."
*Why others wrong:* A) Would violate Atomicity, leaving inconsistent state. C) Retrying isn't an ACID guarantee. D) Statements inside one transaction are not committed individually.

**Q16.** [Hard | Joins] Table A has 3 rows, Table B has 4 rows. A CROSS JOIN between them returns how many rows?
A) 7 B) 12 C) 3 D) 4
**Answer: B**
*Explanation:* CROSS JOIN produces the Cartesian product — every row of A paired with every row of B, i.e., 3 × 4 = 12.
*Why others wrong:* A) That's the sum, not the product. C, D) Reflect only one table's row count.

**Q17.** [Hard | Self Join] Which scenario is a classic use case for SELF JOIN?
A) Joining employee table with department table B) Finding employees who share the same manager, using the employee table joined with itself C) Combining two unrelated tables D) Creating an index
**Answer: B**
*Explanation:* A self join relates a table to itself, commonly used for hierarchical data like employee-manager relationships within the same table.
*Why others wrong:* A) That's a normal INNER JOIN between two distinct tables. C) Self join is specifically about the same table, not unrelated ones. D) Unrelated to indexing.

**Q18.** [Medium | Views] Why might a view based on a JOIN with GROUP BY fail to be updatable?
A) Views never allow SELECT B) Aggregated/grouped views don't map cleanly back to individual base-table rows for updates C) MySQL blocks all views from SELECT D) It's actually always updatable
**Answer: B**
*Explanation:* When a view aggregates or joins data, there's no unambiguous one-to-one mapping back to a single base-table row, so the DBMS restricts UPDATE/INSERT/DELETE through it.
*Why others wrong:* A, C) Views fully support SELECT. D) Complex views are often explicitly non-updatable.

**Q19.** [Medium | Index Trap] Which scenario is LEAST likely to benefit from adding an index?
A) A column frequently used in WHERE clauses on a huge table B) A column with very few distinct values (e.g., gender) on a huge table C) A column used as a JOIN key D) A column used in ORDER BY frequently
**Answer: B**
*Explanation:* Low-cardinality columns (few distinct values) provide little filtering benefit from an index, since the index can't narrow down rows much.
*Why others wrong:* A, C, D) All are classic strong candidates for indexing since they narrow down or order large result sets efficiently.

**Q20.** [Hard | Subquery] What happens if you use `=` with a subquery that returns 2 rows?
A) Returns first row only B) Runtime error: subquery returns more than 1 row C) Returns NULL D) Automatically converts to IN
**Answer: B**
*Explanation:* The `=` operator expects a single scalar value; a multi-row subquery result causes a runtime error.
*Why others wrong:* A, C, D) None of these are actual MySQL behaviors — it throws an explicit error instead.

**Q21.** [Medium | Normalization Trap] Denormalization is typically used to:
A) Enforce stricter data integrity B) Improve read performance by reducing JOINs, at the cost of redundancy C) Remove all indexes D) Convert tables to BCNF
**Answer: B**
*Explanation:* Denormalization intentionally introduces redundancy to reduce the number of JOINs needed, speeding up reads in read-heavy systems.
*Why others wrong:* A) Denormalization typically loosens, not tightens, integrity guarantees. C) Unrelated to indexes. D) BCNF is a stricter normal form, opposite of denormalization's goal.

**Q22.** [Easy | Terminology] ACID stands for:
A) Atomicity, Consistency, Isolation, Durability B) Accuracy, Constraint, Index, Data C) Atomicity, Concurrency, Integrity, Durability D) Availability, Consistency, Isolation, Distribution
**Answer: A**
*Explanation:* ACID = Atomicity, Consistency, Isolation, Durability — the four guarantees of reliable database transactions.
*Why others wrong:* B, C, D are fabricated or mix in unrelated distributed-systems terms (like the CAP theorem's "Availability"/"Consistency").

**Q23.** [Hard | Trap] Which best explains why a LEFT JOIN followed by `WHERE right_table.col IS NULL` is a common pattern?
A) It finds rows that exist only in the left table (anti-join pattern) B) It's a syntax error C) It finds only matching rows D) It's identical to an INNER JOIN
**Answer: A**
*Explanation:* This pattern (LEFT JOIN + IS NULL filter) is the classic way to find rows in the left table that have NO match in the right table — an "anti-join."
*Why others wrong:* B) It's valid, commonly used SQL. C) It specifically excludes matching rows via the IS NULL filter. D) INNER JOIN would return only matches, the opposite intent.

**Q24.** [Medium | Isolation] Which ACID property is most directly responsible for preventing one transaction from seeing another's uncommitted changes?
A) Atomicity B) Consistency C) Isolation D) Durability
**Answer: C**
*Explanation:* Isolation controls how/when changes made by one transaction become visible to other concurrent transactions.
*Why others wrong:* A) Concerns whether a transaction's own steps all succeed or none do. B) Concerns overall valid state. D) Concerns persistence post-commit.

**Q25.** [Hard | Integration] Rank in typical execution order within a single SELECT query: (1) HAVING (2) FROM/JOIN (3) WHERE (4) GROUP BY (5) SELECT (6) ORDER BY
A) 2 → 3 → 4 → 1 → 5 → 6 B) 5 → 2 → 3 → 4 → 1 → 6 C) 2 → 5 → 3 → 4 → 1 → 6 D) 1 → 2 → 3 → 4 → 5 → 6
**Answer: A**
*Explanation:* Logical SQL processing order is FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY, even though we *write* SELECT first syntactically.
*Why others wrong:* B, C, D place SELECT or HAVING out of the true logical evaluation order.

---

## Module 4 (MySQL) Complete ✅
Score yourself: if you missed any of Q3, Q5, Q12, Q18, Q20, Q23, or Q25 across both MySQL sections — reread Joins and Normalization/ACID before moving on, these carry the highest Capgemini trap density.

---

# MODULE 5 — Git: Version Control, Branching, Merge/Rebase, Remotes, Conflicts

## 1. What is Version Control? Why Git?
- **Version Control System (VCS)**: tracks changes to files over time, allows reverting, comparing, and collaborating without overwriting others' work.
- **Git** is a **Distributed** VCS (DVCS) — every developer has a **full copy** of the repository history locally, unlike centralized systems (e.g., old SVN) where history lives only on a central server.

**Capgemini trap:** "Git requires a network connection for commit/log/diff" → **False**. Those are all **local** operations. Only `push`, `pull`, `fetch`, `clone` need network access.

## 2. Git Architecture — THE #1 asked concept

```
Working Directory  →  [git add]  →  Staging Area (Index)  →  [git commit]  →  Local Repository (.git)  →  [git push]  →  Remote Repository
```

| Area | Description |
|---|---|
| **Working Directory** | Your actual files on disk, editable |
| **Staging Area (Index)** | Snapshot of changes marked to be included in the next commit |
| **Local Repository** | Committed history, stored in `.git` folder on your machine |
| **Remote Repository** | Shared copy hosted elsewhere (GitHub/GitLab/Bitbucket) |

**Trap:** `git add` does NOT save history — it only stages. Only `git commit` actually records a snapshot into the local repository's history.

## 3. Core Commands

| Command | Purpose |
|---|---|
| `git init` | Initializes a new local Git repository |
| `git clone <url>` | Copies an existing remote repo (with full history) locally |
| `git status` | Shows staged/unstaged/untracked changes |
| `git add <file>` / `git add .` | Stages changes |
| `git commit -m "msg"` | Records staged changes into local history |
| `git log` | Shows commit history |
| `git diff` | Shows unstaged changes (working dir vs staging/last commit) |

**Trap:** `git diff` alone shows **working dir vs staging area**. `git diff --staged` (or `--cached`) shows **staging area vs last commit**.

## 4. Branching

| Command | Purpose |
|---|---|
| `git branch <name>` | Creates a new branch (doesn't switch to it) |
| `git checkout <name>` | Switches to a branch (older syntax) |
| `git switch <name>` | Switches to a branch (newer, dedicated command) |
| `git checkout -b <name>` | Creates AND switches to a new branch in one step |

- A branch is essentially a **movable pointer** to a commit — cheap and fast to create in Git (unlike some older VCS).
- `main`/`master` is typically the default/primary branch.

## 5. Merge vs Rebase — very frequently asked

| | Merge | Rebase |
|---|---|---|
| History | Preserves full branch history, creates a **merge commit** | Rewrites commit history — replays commits on top of target branch, **linear** history |
| Safety | Safe for shared/public branches | **Never rebase commits already pushed/shared** with others — rewrites history and breaks others' clones |
| Result | Non-linear graph with merge commits | Clean, linear commit log |
| Command | `git merge <branch>` | `git rebase <branch>` |

**Trap:** "Rebase and merge always give the same final file content but differ in **history shape**" — this is largely true; the key exam trap is about **when NOT to rebase** (shared/pushed branches).

## 6. Cherry-pick, Reset, Revert — commonly confused trio

| Command | What it does |
|---|---|
| `git cherry-pick <commit>` | Applies a **specific commit** from one branch onto another (not the whole branch) |
| `git reset` | Moves the branch pointer backward; can discard commits (`--hard`) or keep changes staged/unstaged (`--soft`/`--mixed`) |
| `git revert <commit>` | Creates a **new commit** that undoes a previous commit's changes — safe for shared history |

**Trap:** `git reset --hard` **rewrites history and discards changes** — dangerous on shared branches. `git revert` is the **safe** alternative for shared/public history because it adds a new commit instead of rewriting.

## 7. Stash
`git stash` temporarily shelves uncommitted changes (working directory + staging) so you can switch branches cleanly, then `git stash pop` restores them.

## 8. Remotes: origin, push, pull, fetch

| Command | Purpose |
|---|---|
| `git remote add origin <url>` | Links local repo to a remote named "origin" |
| `git push origin <branch>` | Uploads local commits to remote |
| `git fetch` | Downloads remote changes **without** merging into your working branch |
| `git pull` | `git fetch` + `git merge` combined — downloads AND merges |

**Trap:** `git fetch` is **safe** (just downloads, doesn't touch your working files). `git pull` can trigger merge conflicts immediately since it auto-merges.

## 9. Conflict Resolution
Conflicts happen when Git can't automatically reconcile changes to the **same lines** in the same file across branches.
```
<<<<<<< HEAD
your current branch's version
=======
incoming branch's version
>>>>>>> feature-branch
```
- Manually edit the file to keep the correct content, remove the conflict markers, then `git add <file>` and `git commit` (for merge) or `git rebase --continue` (for rebase).

## One-Page Revision (Git)
- Git = distributed VCS; commit/log/diff are local (no network needed); push/pull/fetch/clone need network.
- Flow: Working Dir → (add) → Staging → (commit) → Local Repo → (push) → Remote Repo.
- `git diff` = working vs staging. `git diff --staged` = staging vs last commit.
- Branch = movable pointer to a commit. `checkout -b` = create + switch in one command.
- Merge = preserves history, creates merge commit. Rebase = linear history, rewrites commits — never rebase shared/pushed commits.
- Cherry-pick = apply one specific commit elsewhere. Reset = move pointer/discard (dangerous, local). Revert = new commit undoing a past commit (safe, shareable).
- Fetch = download only (safe). Pull = fetch + merge (can conflict immediately).
- Conflict markers: `<<<<<<<`, `=======`, `>>>>>>>` — resolve manually, then add + commit/continue.

---

# MCQs — Git: Version Control, Branching, Merge/Rebase, Remotes, Conflicts (25 Questions)

**Q1.** [Easy | VCS Basics] Which of the following Git commands does NOT require a network connection?
A) git push B) git clone C) git commit D) git pull
**Answer: C**
*Explanation:* git commit operates entirely on the local repository, recording staged changes into local history.
*Why others wrong:* A, D) Require syncing with a remote server. B) Requires downloading the repo from a remote source.

**Q2.** [Easy | Architecture] What does `git add` do?
A) Permanently commits changes to history B) Stages changes into the index for the next commit C) Pushes changes to remote D) Creates a new branch
**Answer: B**
*Explanation:* git add moves changes from the working directory into the staging area, preparing them for the next commit.
*Why others wrong:* A) That's git commit's job. C) That's git push's job. D) That's git branch's job.

**Q3.** [Medium | Diff Trap] Plain `git diff` (no flags) compares:
A) Staging area vs last commit B) Working directory vs staging area C) Local repo vs remote repo D) Two arbitrary commits
**Answer: B**
*Explanation:* Without flags, git diff shows unstaged changes — the difference between your working directory and what's currently staged.
*Why others wrong:* A) That's `git diff --staged`. C) That would need explicit remote comparison syntax. D) That would need `git diff <commit1> <commit2>`.

**Q4.** [Medium | Branching] What is a Git branch, conceptually?
A) A full copy of the entire repository B) A movable pointer to a specific commit C) A backup file D) A remote server
**Answer: B**
*Explanation:* A branch is simply a lightweight, movable reference pointing to a commit, which is why branching in Git is fast and cheap.
*Why others wrong:* A) That's closer to a clone. C) Not what a branch represents. D) Unrelated concept.

**Q5.** [Easy | Branching] Which command creates AND switches to a new branch in a single step?
A) `git branch -new` B) `git checkout -b <name>` C) `git switch <name>` D) `git merge <name>`
**Answer: B**
*Explanation:* `git checkout -b <name>` combines branch creation and switching into one command.
*Why others wrong:* A) Not valid Git syntax. C) Switches to an EXISTING branch, doesn't create one (unless combined with -c). D) Merge combines branch histories, doesn't create/switch branches.

**Q6.** [Medium | Merge vs Rebase] Which statement about `git merge` is TRUE?
A) It rewrites commit history into a linear log B) It preserves full branch history and creates a merge commit C) It deletes the source branch automatically D) It requires network access
**Answer: B**
*Explanation:* Merge integrates changes from one branch into another while preserving both branches' commit history, typically via a merge commit.
*Why others wrong:* A) That describes rebase, not merge. C) Merging doesn't auto-delete branches. D) Merge is a local operation.

**Q7.** [Medium | Merge vs Rebase] Which statement about `git rebase` is TRUE?
A) It preserves a non-linear history graph B) It replays commits on top of another branch, creating a linear history C) It's always safe on shared/pushed branches D) It doesn't affect commit hashes
**Answer: B**
*Explanation:* Rebase takes your branch's commits and reapplies them on top of the target branch's tip, producing a clean, linear history.
*Why others wrong:* A) That's merge's behavior. C) Rebasing shared/pushed commits is dangerous — it rewrites history others depend on. D) Rebase creates new commits with new hashes, since the parent changes.

**Q8.** [Hard | Trap] Why is it risky to rebase commits that have already been pushed and shared with teammates?
A) Rebase is always slower than merge B) It rewrites commit history/hashes, causing conflicts and confusion for anyone who already pulled the old commits C) It permanently deletes the remote repository D) It automatically force-pushes without warning
**Answer: B**
*Explanation:* Rebase creates new commits with new hashes to replace the old ones; anyone who already has the old commits will face diverging/duplicate history when they try to sync.
*Why others wrong:* A) Performance isn't the core concern. C) Rebase doesn't delete the remote repo. D) Rebase itself doesn't auto-push; a manual force-push is a separate, additional risky step.

**Q9.** [Medium | Reset vs Revert] Which command creates a NEW commit that undoes a previous commit's changes, making it safe for shared branches?
A) git reset --hard B) git revert C) git cherry-pick D) git stash
**Answer: B**
*Explanation:* git revert adds a new commit that reverses the changes of a target commit, preserving history — safe to use on shared/public branches.
*Why others wrong:* A) Reset rewrites/discards history directly, risky if shared. C) Cherry-pick applies a commit elsewhere, doesn't undo one. D) Stash temporarily shelves changes, doesn't undo commits.

**Q10.** [Medium | Cherry-pick] What does `git cherry-pick <commit-hash>` do?
A) Deletes a specific commit B) Applies a specific commit's changes onto the current branch C) Merges two entire branches D) Reverts the entire branch history
**Answer: B**
*Explanation:* Cherry-pick takes one specific commit from elsewhere and applies just that commit's changes onto your current branch.
*Why others wrong:* A) It doesn't delete anything. C) That's what merge does (whole branches, not single commits). D) That's not revert's/cherry-pick's scope.

**Q11.** [Hard | Reset Trap] Which reset mode discards changes from both the staging area AND working directory?
A) git reset --soft B) git reset --mixed C) git reset --hard D) git reset --keep
**Answer: C**
*Explanation:* `--hard` resets the branch pointer and wipes both the staging area and working directory changes, making it the most destructive reset mode.
*Why others wrong:* A) --soft only moves the pointer, keeps changes staged. B) --mixed unstages changes but keeps them in the working directory. D) Not a commonly tested standard flag in this context.

**Q12.** [Easy | Remotes] What does `origin` typically refer to in Git?
A) The very first commit in a repo B) The default name for the primary remote repository C) A type of branch D) A merge conflict marker
**Answer: B**
*Explanation:* "origin" is simply the conventional default name Git assigns to the remote repository a project was cloned from.
*Why others wrong:* A) That's unrelated to commit numbering. C) It's a remote name, not a branch type. D) Conflict markers are `<<<<<<<`, `=======`, `>>>>>>>`.

**Q13.** [Medium | Fetch vs Pull] What is the key difference between `git fetch` and `git pull`?
A) fetch downloads AND merges, pull only downloads B) fetch only downloads remote changes; pull downloads AND merges them C) They are identical commands D) fetch requires no network, pull does
**Answer: B**
*Explanation:* git fetch retrieves remote commits without touching your working branch, while git pull performs a fetch followed by an automatic merge.
*Why others wrong:* A) Reverses the actual behavior. C) They behave differently, not identically. D) Both require network access to reach the remote.

**Q14.** [Medium | Safety Trap] Why is `git fetch` considered "safer" than `git pull` for inspecting remote changes first?
A) Fetch is faster over slow networks B) Fetch downloads changes without automatically merging, letting you review before integrating C) Fetch doesn't require authentication D) Fetch only works on the main branch
**Answer: B**
*Explanation:* Since fetch doesn't auto-merge, you can inspect incoming changes (e.g., via `git log` or `git diff`) before deciding to merge, avoiding surprise conflicts.
*Why others wrong:* A) Speed isn't the core safety reason. C) Both fetch and pull require the same authentication to the remote. D) Fetch works on any branch, not just main.

**Q15.** [Easy | Stash] What does `git stash` do?
A) Permanently deletes uncommitted changes B) Temporarily shelves uncommitted changes so you can switch branches cleanly C) Pushes changes to remote D) Creates a new commit
**Answer: B**
*Explanation:* Stash saves your uncommitted working directory and staging changes aside, giving you a clean working directory, which you can later restore with `git stash pop`.
*Why others wrong:* A) Changes aren't deleted, just shelved (recoverable). C) Stash is entirely local. D) Stash explicitly avoids committing incomplete work.

**Q16.** [Medium | Conflicts] Conflict markers `<<<<<<<`, `=======`, `>>>>>>>` appear when:
A) A commit message is invalid B) Git cannot automatically reconcile changes to the same lines across branches C) You run git status D) You delete a branch
**Answer: B**
*Explanation:* These markers appear in a file when Git detects that both branches modified overlapping lines and needs manual resolution.
*Why others wrong:* A) Unrelated to commit message validity. C) git status doesn't insert conflict markers, it just reports conflict state. D) Deleting a branch doesn't cause file conflicts.

**Q17.** [Medium | Conflicts] After manually resolving a merge conflict in a file, what's the next step?
A) git commit directly without staging B) git add <file>, then git commit C) git push immediately D) git reset --hard
**Answer: B**
*Explanation:* After editing the file to resolve markers, you must stage it with git add to mark the conflict as resolved, then commit to finalize the merge.
*Why others wrong:* A) You must stage the resolved file first. C) Pushing isn't required to complete a local merge. D) That would discard your resolution work.

**Q18.** [Hard | Scenario] You're mid-rebase and hit a conflict. After manually fixing it and staging the file, which command continues the rebase?
A) git commit B) git rebase --continue C) git merge --continue D) git push --force
**Answer: B**
*Explanation:* During a rebase, after resolving a conflict and staging the fix, `git rebase --continue` resumes replaying the remaining commits.
*Why others wrong:* A) Rebase has its own continuation flow, not a plain commit. C) That syntax applies to merge conflicts, not rebase. D) Force-push is a separate, unrelated remote operation.

**Q19.** [Medium | Distributed VCS] What makes Git a "distributed" version control system?
A) Code is distributed across multiple programming languages B) Every developer has a full local copy of the repository, including history C) It only works with cloud servers D) It distributes tasks across a team automatically
**Answer: B**
*Explanation:* In Git, each clone contains the entire project history locally, unlike centralized systems where history lives only on a central server.
*Why others wrong:* A) Unrelated to programming languages. C) Git works fully offline with local repos; cloud isn't required. D) Git doesn't do task management/distribution.

**Q20.** [Medium | Log/History] Which command shows the commit history of a repository?
A) git status B) git log C) git diff D) git branch
**Answer: B**
*Explanation:* git log displays the chronological list of commits, including hashes, authors, dates, and messages.
*Why others wrong:* A) Shows current working/staging state, not history. C) Shows differences, not a commit list. D) Lists branches, not commit history.

**Q21.** [Hard | Trap] Which statement about `git clone` vs `git init` is TRUE?
A) `git init` downloads an existing remote repo; `git clone` starts a new empty one B) `git clone` copies an existing remote repo with full history; `git init` starts a brand-new empty local repo C) Both do exactly the same thing D) `git init` requires network access
**Answer: B**
*Explanation:* clone duplicates an existing remote repository (code + full history) to your machine, while init creates a fresh, empty repository from scratch.
*Why others wrong:* A) Reverses the actual behaviors. C) They serve different purposes. D) init is purely local and needs no network.

**Q22.** [Medium | Scenario] Two teammates both edited line 10 of the same file on different branches, then tried to merge. What is the most likely outcome?
A) Git silently picks one version at random B) A merge conflict requiring manual resolution C) Both versions are automatically combined into one line D) The merge fails permanently with no recovery
**Answer: B**
*Explanation:* When the same lines are changed differently on both branches, Git cannot auto-resolve and flags a conflict for manual intervention.
*Why others wrong:* A) Git never resolves conflicting line edits randomly. C) Auto-combination only works for non-overlapping changes, not the same line. D) Conflicts are always recoverable by resolving and committing.

**Q23.** [Easy | Terminology] What does "checkout" traditionally do in Git?
A) Pays for a Git subscription B) Switches your working directory to a different branch or commit C) Deletes a branch D) Pushes changes to remote
**Answer: B**
*Explanation:* git checkout changes what your working directory reflects — switching branches or restoring specific commit states.
*Why others wrong:* A) Not a real Git concept. C) Deleting uses `git branch -d`. D) Unrelated to pushing.

**Q24.** [Medium | Best Practices] Which is considered Git best practice for commit messages and workflow?
A) Commit rarely, with huge unrelated changes bundled together B) Make small, focused commits with clear, descriptive messages C) Always force-push to shared branches to keep history clean D) Never use branches, always commit directly to main
**Answer: B**
*Explanation:* Small, focused commits with descriptive messages make history easier to read, review, revert, and debug (e.g., via git bisect).
*Why others wrong:* A) Large bundled commits make history hard to review/revert. C) Force-pushing to shared branches is dangerous and generally discouraged. D) Direct commits to main bypass code review and isolation benefits of branching.

**Q25.** [Hard | Integration] Order these Git operations by their typical workflow sequence for a new feature: (1) git push origin feature (2) git checkout -b feature (3) git commit -m "add feature" (4) git add . (5) create Pull Request
A) 2 → 4 → 3 → 1 → 5 B) 4 → 2 → 3 → 1 → 5 C) 2 → 3 → 4 → 1 → 5 D) 5 → 2 → 4 → 3 → 1
**Answer: A**
*Explanation:* Standard flow: create/switch to a feature branch (2) → stage changes (4) → commit (3) → push to remote (1) → open a Pull Request for review (5).
*Why others wrong:* B, C, D scramble the logical sequence — you can't stage before branching, commit before staging, or PR before pushing.

---

## Module 5 (Git) Complete ✅
Score yourself: if you missed any of Q7, Q8, Q9, Q11, Q18, Q21, or Q25 — reread Merge vs Rebase and Reset vs Revert before your exam, these are the highest-trap-density Git questions at Capgemini L1.

---

# Combined Revision Snapshot (MySQL Modules 4 + Git Module 5)

| Topic | One-line memory hook |
|---|---|
| TRUNCATE | DDL, not DML — no WHERE, resets identity, not rollback-able |
| PRIMARY KEY vs UNIQUE | PK = no NULL, only 1 per table; UNIQUE allows NULL, multiple allowed |
| WHERE vs HAVING | WHERE filters rows before grouping; HAVING filters groups after |
| LEFT JOIN | All left rows + matched right (NULL if no match) |
| FULL JOIN in MySQL | Not native — simulate via UNION of LEFT + RIGHT JOIN |
| Subquery `=` vs `IN` | `=` needs single value; `IN` for multiple rows |
| Views | Virtual table from a query; complex views often not updatable |
| Indexes | Faster reads, slower writes, more storage |
| Normalization | 1NF atomic → 2NF no partial dependency → 3NF no transitive dependency |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Git local vs network | commit/log/diff/add = local; push/pull/fetch/clone = network |
| Merge vs Rebase | Merge preserves history; Rebase rewrites it — never rebase shared commits |
| Reset vs Revert | Reset rewrites/discards (risky, local); Revert adds new undo commit (safe, shareable) |
| Fetch vs Pull | Fetch = download only; Pull = fetch + auto-merge |

---

**Next up:** Say "next chapter" to continue to further modules (Spring & Spring Boot, Frontend, or Mock Tests) whenever you're ready.
