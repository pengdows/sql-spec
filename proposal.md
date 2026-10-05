# [INCITS document number to be assigned] / SQL:2023 — Proposal for a New Optional Feature T4xx: Include Columns in Data Change Delta Tables

**Document number:** To be assigned by INCITS Data Management<br>
**Date:** Sep 28, 2026<br>
**Author:** Alaric [full name to be supplied]<br>
**Submitted by:** Alaric, on behalf of the Pengdows project<br>
**Status:** Draft proposal to extend feature T495 (Combined data change and retrieval) with declared pass-through columns for INSERT, UPDATE, and MERGE. The core semantics are settled; final standards integration and the explicitly marked provisional rules await two research gates: the disposition of ballot comment P02-USA-200 and verification against ISO/IEC 9075-2:2023.

When INCITS assigns the document number and feature number, replace the placeholders in this heading and metadata. The subfeature identifier and name, currently T4xx-01, must be updated at the same time.

Syntax, Syntax rules, General rules, and Conformance are normative proposal text, subject to the provisional qualifications explicitly stated below. Every provisional rule is pending verification against the relevant ISO/IEC 9075-2:2023 Part 2 grammar, Syntax Rules, General Rules, and Access Rules; Annex F is used only for feature classification. Everything from Exclusions onward is rationale, research, and working notes, kept separate so explanatory prose doesn't leak into the rules.

## Abstract

This proposal defines an optional SQL feature, "Include columns in data change delta tables," requiring feature T495, "Combined data change and retrieval." The feature permits an `INSERT`, `UPDATE`, or eligible `MERGE` statement contained in a data change delta table to declare typed, nullable include columns. Include columns are appended to the row type of the data change delta table and can be assigned while processing the data change statement. They are not columns of the target table or view and are not stored.

The feature enables portable correlation between changed target rows and operation-local values, including source correlation keys, pre-update values, and branch-specific `MERGE` values. It does not add ordering, source-row identity, or identity-contiguity guarantees.

## Problem

A data change delta table returns only target-row images. Values that take part in a data change but aren't stored in the target are lost:

- **INSERT … SELECT:** source columns that aren't inserted, such as a correlation key.
- **UPDATE:** auxiliary values, such as the pre-update value next to the new row.
- **MERGE:** source-row values and values specific to the branch that fired.

As a result there's no reliable way to correlate generated keys with the source rows that produced them. Row order and identity contiguity aren't guaranteed, so neither can stand in for correlation.

## Background

This proposal arose from implementation experience rather than from a theoretical extension to the language. While developing `pengdows.flatfile`, a standards-oriented, file-backed SQL engine, and testing it through `pengdows.crud`, a SQL execution layer for .NET, I encountered the need to correlate rows supplied to a data change statement with the rows returned from that change.

Investigation showed that several database systems provide vendor-specific mechanisms addressing parts of this problem, but there is no portable SQL mechanism that carries non-target values through a data change delta table. After finding that T495 already provides the relational foundation, and that P02-USA-200 had previously identified the missing INCLUDE capability without supplying proposal text, this proposal was developed to fill that gap while preserving the existing T495 model.

## Syntax

```text
<include column clause> ::=
  INCLUDE <left paren> <include column definition>
    [ { <comma> <include column definition> }... ] <right paren>

<include column definition> ::= <column name> <data type>
```

Where the clause attaches:

```text
INSERT INTO t [ <insert column list> ] [ <include column clause> ] <insert source>
UPDATE t [ [ AS ] c ] [ <include column clause> ] SET ... [ WHERE ... ]
MERGE INTO t [ [ AS ] c ] [ <include column clause> ] USING ... ON ... <when clauses>
```

The INSERT schematic intentionally uses the standard `<insert column list>` production rather than expanding it. Its exact placement relative to any `<override clause>` is provisional pending the Gate 2 grammar walk. `INCLUDE` is a non-reserved word *(provisional; see Provisional rules)*.

An **include column** is a column of the row type of a data change delta table that is declared by an `<include column clause>`, is not a column of the target table or view, and exists only for evaluation of the containing data change delta table.

## Syntax rules

1. **Delta tables only.** An `<include column clause>` may appear only when the data change statement is directly contained in a `<data change delta table>`.
2. **Unique names.** Include column names must be distinct from each other and from every column of the target table or view.
3. **Definitions.** An include column definition is a name and a data type, with no constraints and no default.
4. **Data types.** **Provisional, pending verification against ISO/IEC 9075-2:2023 Part 2:** Without feature T4xx-01, the declared type of an include column shall not be a LOB type, the XML type, `LONG VARCHAR`, `LONG VARGRAPHIC`, or a distinct type based on one of those types. Annex F is relevant only to the classification of T4xx-01. The final restricted type set and feature boundary remain subject to the Part 2 review.
5. **INSERT degree.** If the insert source supplies an explicit row value, the number of values in each supplied row shall equal the number of columns in the explicit or implicit insert column list plus the number of include columns declared by the `<include column clause>`. Values corresponding to include columns follow the values corresponding to target columns, in declaration order. If an implicit insert column list is used, its membership and ordering are determined entirely by the existing rules for an omitted insert column list; include columns are not members of that implicit list.
6. **SET assignment.** A `<set clause>` in an UPDATE statement or MERGE update action may assign an include column. Each include column is assigned at most once per `<set clause list>`. An UPDATE statement or MERGE update action that assigns an include column shall contain at least one assignment to a column of the target table or view.
7. **MERGE insert.** **Provisional, pending verification against the SQL:2023 Part 2 MERGE grammar and Syntax Rules:** A `<merge insert specification>` column list may name include columns. A value in an include column's position assigns that column and isn't inserted into the target. Each include column may be named at most once in the column list. A `<merge insert specification>` that names an include column shall also name at least one column of the target table or view.
8. **Contextual typing.** **Provisional, pending verification against the SQL:2023 Part 2 grammar and Syntax Rules:** Where a value specification occurs in a position corresponding to an include column, that position has the declared data type of the include column for purposes of contextual typing. An include column definition has no default. `DEFAULT` in such a position yields the null value of that declared type.
9. **Assignment only.** An include column can't be referenced except as an assignment target and through the containing query's use of the delta table. In particular, it can't appear on the right-hand side of an assignment. **The requirement that an include-column assignment target be specified without qualification is provisional, pending verification against the SQL:2023 UPDATE and MERGE assignment-target productions.**
10. **Exclusions.** Not permitted on a `<delete statement>`, or on a `<merge statement>` that contains a `<merge delete specification>`. **The MERGE-with-DELETE interaction is also subject to verification against the existing SQL:2023 MERGE and delta-table rules.**

## General rules

1. **Row type.** The delta table's row type is the target's row type (the view's, for a view target) followed by the include columns, in declaration order.
2. **Assignment.** A value assigned to an include column is assigned according to the existing General Rules for store assignment, with the include column as the target. *(Provisional; the subclause number and exact cross-reference must be verified against SQL:2023.)*
3. **Values and nullability.** Include columns are nullable. For each row of the delta table, an include column contains the value assigned while producing that row. If the action that produced the row doesn't assign that include column, its value is null. `DEFAULT VALUES` assigns the null value to every include column. `DEFAULT` in an include-column position assigns the null value of the declared type.
4. **Scope and visibility.** Include-column values are evaluated as part of the data change statement and are available only in the corresponding row of the data change delta table. They are not columns of the target table or view and do not constitute insertion into, update of, or deletion from the target. They do not participate in the evaluation of target-table constraints, assertions, generated-column expressions, referential constraints, transition variables, transition tables, referential actions, or triggered SQL statements.
5. **Privileges.** **Provisional, pending verification against SQL:2023 access rules:** The presence of an `<include column clause>` introduces no additional privilege requirement. Existing privilege requirements for the target, referenced objects, expressions, and the containing query are unchanged.
6. **Assignment evaluation unchanged.** Right-hand sides in `SET` see pre-update values.
7. **Existing semantics unchanged.** Only changed rows appear. There are no ordering, contiguity, or uniqueness guarantees. Existing rules on view targets and delta tables still apply.

### Result-option semantics

Subject to the applicability of the result option to the contained data change statement under existing rules, `OLD`, `NEW`, and `FINAL` determine the values of the columns corresponding to the target table or view. Include values are local to the operation and are independent of the result option. This rule applies equally when the target is a view, subject to the existing rules for view targets and data change delta tables.

Character include columns follow the existing collation-derivation rules for their declared character type; this proposal introduces no separate collation rule. That interaction remains subject to verification against the SQL:2023 text.

## Examples

### Generated-key correlation

```sql
SELECT request_id, customer_id
FROM FINAL TABLE (
  INSERT INTO customer (name)
    INCLUDE (request_id CHAR(36))
  SELECT i.name, i.request_id
  FROM incoming_customer AS i
);
```

### Old and new values in one row

```sql
SELECT sku, old_price, price
FROM FINAL TABLE (
  UPDATE product
    INCLUDE (old_price DECIMAL(10,2))
  SET old_price = price, price = price * 1.1
  WHERE category = 'X'
);
```

### MERGE source correlation

```sql
SELECT request_id, id
FROM FINAL TABLE (
  MERGE INTO customer AS c
    INCLUDE (request_id CHAR(36))
  USING incoming AS s ON c.email = s.email
  WHEN MATCHED THEN UPDATE SET name = s.name, request_id = s.request_id
  WHEN NOT MATCHED THEN INSERT (email, name, request_id)
    VALUES (s.email, s.name, s.request_id)
);
```

### MERGE branch-specific null fill-in

```sql
SELECT correlation_id, previous_status, id, status
FROM FINAL TABLE (
  MERGE INTO work_item AS t
    INCLUDE (
      correlation_id CHAR(36),
      previous_status VARCHAR(20)
    )
  USING staging_work_item AS s
    ON t.external_key = s.external_key
  WHEN MATCHED THEN
    UPDATE SET previous_status = t.status,
               status = s.status,
               correlation_id = s.correlation_id
  WHEN NOT MATCHED THEN
    INSERT (external_key, status, correlation_id)
    VALUES (s.external_key, s.status, s.correlation_id)
);
```

For inserted rows, `previous_status` is null because no insert assignment supplies it.

### DEFAULT VALUES yields a null include value

```sql
SELECT id, request_id
FROM FINAL TABLE (
  INSERT INTO t
    INCLUDE (request_id CHAR(36))
  DEFAULT VALUES
);
```

## Conformance

- **T4xx, "Include columns in data change delta tables":** a new optional feature (number to be assigned). Requires T495. The title and every T4xx reference in this proposal must be updated when INCITS assigns the feature number.
- **T4xx-01, "Include columns of LOB, XML, and long character types":** provisionally lifts the type restriction in Syntax Rule 4. Requires T4xx. The final restricted type set, title, feature classification, and dependency wording are subject to verification against SQL:2023 Part 2 and Annex F.
- **MERGE with DELETE:** T4xx does not introduce a separate conformance claim for MERGE delete branches. Its applicability to a MERGE containing a delete branch remains subject to verification against the existing SQL:2023 MERGE and delta-table rules.

## Exclusions and rationale

- **Standalone DELETE.** Excluded from the initial feature. It doesn't address the source-correlation problem, because standard DELETE has no independent source relation, and it would require a new assignment clause on DELETE. Db2 permits INCLUDE on DELETE. Adding it later wouldn't change the semantics defined here.
- **MERGE with a DELETE action.** The new delta table of a MERGE is the union of its INSERT and UPDATE delta tables, so deleted rows have no image in it. Db2 likewise rejects a MERGE containing a delete operation when it is used as a data-change table (SQLCODE -270). The rule rejects the statement rather than silently dropping rows, and can be relaxed later without breaking existing code.
- **Action pseudo-column.** Out of scope. The same information is available by assigning an include column in each branch.

The base type restriction addresses implementation-defined lifetime and materialization semantics for values that can be locator-based or externally represented. An include-column value must remain available in the data change delta table after evaluation of the target-row modification while not becoming a stored attribute of that target row. Locator-based or deferred values can require lifetime, materialization, ownership, and resource-management semantics beyond ordinary scalar assignment. T4xx-01 permits an implementation to provide those semantics.

## Prior art

| Item | Source | Status |
| --- | --- | --- |
| INCLUDE on INSERT, UPDATE, DELETE, MERGE | Db2 | Demonstrated |
| Include columns appended to the intermediate result row | Db2 | Demonstrated |
| Include columns nullable; unassigned values null | [Db2 for z/OS (MERGE)](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=statements-merge) | Demonstrated |
| DEFAULT for an include-column position yields null | Db2 for z/OS MERGE | Demonstrated |
| Omitted include column in a MERGE insert list yields null | Db2 for z/OS MERGE | Demonstrated |
| LONG VARCHAR, LONG VARGRAPHIC, LOB, XML, and derived distinct types disallowed | [Db2 for z/OS (MERGE)](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=statements-merge) | Demonstrated |
| MERGE with delete rejected as a data-change table | Db2 (SQLCODE -270) | Demonstrated |
| Include-only UPDATE or INSERT assignment rejected | [Db2 for z/OS (SQLCODE -20260, SQLSTATE 428G5)](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=codes-20260) | Demonstrated |
| INCLUDE combined with OLD TABLE | Derived from two documented Db2 rules | Design-derived, no example found |
| MERGE returning source and target rows | [PostgreSQL 17+ MERGE … RETURNING](https://www.postgresql.org/docs/18/sql-merge.html); [SQL Server OUTPUT](https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-ver17) | Related mechanism |
| INSERT … SELECT returning source columns | Not supported by PostgreSQL, SQLite, MariaDB RETURNING; Oracle disallows RETURNING there | Gap |

Db2 also rejects UPDATE and MERGE UPDATE actions that assign only include columns, and MERGE insert lists that contain only include columns. Its [implicit insert column list](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=statements-insert) covers every column not defined as implicitly hidden (INSERT). Row-image semantics are corroborated by [jOOQ's documentation](https://www.jooq.org/doc/latest/manual/sql-building/table-expressions/data-change-delta-tables/): OLD is invalid for INSERT, NEW and FINAL are invalid for DELETE.

## Standards history

Comment P02-USA-200 (DM32.2-2013-00032R2, comment 22, Major Technical) identified a closely related requirement: communicating additional information from a data change delta table, including which MERGE branch fired. It supplied no solution text. The disposition and subsequent treatment remain under review. This proposal does not claim to implement an adopted disposition of that comment.

## Provisional rules

These fill gaps an implementer would otherwise hit. **Every item in this table is provisional and pending verification against the relevant ISO/IEC 9075-2:2023 Part 2 grammar, Syntax Rules, General Rules, and Access Rules. Annex F is relevant only to feature classification.** The table is not intended to imply that the cited wording is already present in, or settled by, SQL:2023.

**Correlation-name parsing note.** No special disambiguation rule is proposed. In the intended `UPDATE` and `MERGE` placements, `INCLUDE <left paren>` is distinguishable from a correlation-name continuation by the following token sequence. This conclusion remains subject to the SQL:2023 grammar walk, including any intervening production not visible in earlier public grammars.

| Rule | Fills | Rationale | Confidence |
| --- | --- | --- | ---: |
| General rule 2: store assignment | How a value becomes the declared type | Reuses existing assignment rules; truncation and cast errors match target columns; one implementation path | 0.85 |
| Syntax rule 8: contextual typing | Types for NULL and DEFAULT in include positions | Uses the include column's declared type; DEFAULT maps to null without importing unrelated target-column semantics | 0.85 |
| General rule 5: privileges | Whether assigning needs a privilege | The clause introduces no additional privilege requirement; existing access rules continue to govern evaluated objects and expressions | 0.8 |
| Syntax rule 4 and T4xx-01: data types | Whether the restricted base type set and subfeature boundary are appropriate | Db2 supplies strong vendor precedent; locator lifetime and materialization semantics provide an implementation rationale, but the SQL:2023 Part 2 provisions must establish the final boundary | 0.7 |
| Correlation-name parsing note | Whether `INCLUDE <left paren>` is distinguishable in the intended UPDATE and MERGE placements | No special disambiguation rule is proposed; verify intervening SQL:2023 productions and key-word treatment | 0.75 |

Collation of character include columns follows `<column definition>` rules for the declared type, with implicit derivation. No new rule is proposed (0.7). Assignment-target qualification and the requirement that an include-bearing MERGE insert also name at least one target column are provisional pending the SQL:2023 grammar check.

## Research gates

### Gate 1: P02-USA-200 disposition

Inquiry sent to the INCITS Data Management secretariat. The outcome decides the framing:

The gate deliverable is the exact comment text, ballot-cycle identification, WG/DM disposition record, committee response or disposition category, any successor paper or meeting record, and confirmation of whether the comment was handled editorially, technically rejected, deferred, or superseded by T495 wording.

- Rejected: answer the recorded objection.
- Deferred pending a paper: this proposal is that paper.
- Accepted: trace where it disappeared in later drafts.
- No disposition found: cite it as prior recognition and submit a new proposal.

### Gate 2: ISO/IEC 9075-2:2023 verification

- Feature list scan, by proxy: PostgreSQL's [supported](https://www.postgresql.org/docs/18/features-sql-standard.html) and [unsupported](https://www.postgresql.org/docs/18/unsupported-features-sql-standard.html) SQL:2023 lists show only T491, T495, and T501 in that range; no delta-table extension (0.8). Replace the proxy with an actual Annex F check when the 2023 text is available.
- Walk the full grammar chain for delta tables, INSERT, UPDATE, and MERGE, including shared productions. Verify the exact UPDATE/MERGE correlation-name grammar, whether `INCLUDE` is reserved or non-reserved, the standard terminology and rules for an omitted INSERT column list, the placement of the include clause relative to any `<override clause>`, assignment-target qualification, and the productions governing contextual typing. Public grammars stop at SQL:2003, which predates T495, so this needs the standard.
- Read the normative MERGE and delta-table rules for an existing MERGE+DELETE restriction.
- Check whether INCLUDE is in the 2023 key-word list, whether it is reserved or non-reserved, and whether the grammar already disambiguates `INCLUDE <left paren>` without a special Syntax Rule.
- Verify the subclause number and exact rules for store assignment; check existing Access Rules, collation derivation, datatype feature dependencies, and whether assignment-target qualification is already enforced by the grammar.
- Check ISO/IEC 9075-2:2023/Cor 1:2026 for changes affecting `<data change delta table>`, `INSERT`, `UPDATE`, or `MERGE`.
- Check any other amendment, in-ballot, or current-draft change touching those productions.

## Revision notes

| Decision | Reason |
| --- | --- |
| Relation-based design, not a session-state `SELECT GENERATED IDENTITIES` | Session state is ambiguous across statements, triggers, and multi-insert MERGE |
| Source correlation, not ordinals | An ordinal over an unordered source is as arbitrary as execution order |
| Extend T495 rather than add `RETURNING` | T495 is already standard; RETURNING is not |
| Declared types, no inference | MERGE branches and DELETE have no single expression to infer from |
| No `GENERATED IDENTITY` keyword | Ordinary projection covers identities, defaults, generated identifiers, and composite keys |
| Delta table holds changed rows only | Skipped rows can be reconciled against caller-held source keys when those keys are unique |
| DELETE and MERGE+DELETE excluded | DELETE has no independent source values to preserve and requires new assignment syntax; MERGE DELETE has no corresponding NEW row image |
| Result option governs target columns only | Include values are operation-local, which resolves OLD/NEW/FINAL and views |
| `DEFAULT VALUES` and explicit `DEFAULT` allowed, yielding null | Prohibiting an orthogonal case buys nothing |
| Include-only assignment lists prohibited | Matches Db2 -20260; keeps UPDATE and MERGE INSERT consistent; can be relaxed later |
