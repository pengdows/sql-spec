# SQL standard alignment

This file tracks the work needed to turn the semantic proposal in [`proposal.md`](../proposal.md) into editor-ready ISO/IEC 9075-2 change instructions.

## Gate 2 checklist

- [ ] Inspect the actual Annex F feature list rather than relying on PostgreSQL's SQL:2023 feature-list proxy.
- [ ] Walk the full grammar chain for `<data change delta table>`, INSERT, UPDATE, MERGE, and all shared productions.
- [ ] Verify the exact UPDATE and MERGE correlation-name grammar.
- [ ] Determine whether the grammar already disambiguates `INCLUDE <left paren>` without a special Syntax Rule.
- [ ] Verify the SQL standard's exact terminology and rules for an omitted INSERT column list.
- [ ] Verify the placement of `<include column clause>` relative to the INSERT `<override clause>` and the standard `<insert column list>` production.
- [ ] Verify the exact production(s) governing `<contextually typed value specification>` in the affected positions.
- [ ] Verify the normative MERGE and delta-table rules for MERGE actions containing DELETE.
- [ ] Verify whether `INCLUDE` is already a SQL key word and, if so, whether it is reserved or non-reserved.
- [ ] Verify the current subclause number and exact rules for store assignment.
- [ ] Inspect existing Access Rules before freezing the provisional privilege rule.
- [ ] Verify existing collation derivation rules for declared include-column character types.
- [ ] Verify the existing SQL:2023 data-type applicability rules for data change delta tables and the Part 2 basis for the LOB/XML base-feature restriction.
- [ ] Verify the T4xx-01 dependency and Annex F classification; do not infer the standard's boundary solely from vendor behavior.
- [ ] Verify whether assignment-target qualification is already prohibited by the existing grammar.
- [ ] Obtain ISO/IEC 9075-2:2023/Cor 1:2026 (published or current ballot text) and check it for changes affecting `<data change delta table>`, `INSERT`, `UPDATE`, or `MERGE`.
- [ ] Check any other amendment, in-ballot, or current-draft change touching those productions.
- [ ] Produce exact edit instructions against the committee's current working draft.
- [ ] Replace placeholder feature identifiers T4xx/T4xx-01 with committee-assigned identifiers.

## Public proxy evidence

Until the normative text is available, PostgreSQL's SQL-standard feature tables are being used only as a proxy scan:

- [Supported SQL-standard features](https://www.postgresql.org/docs/18/features-sql-standard.html)
- [Unsupported SQL-standard features](https://www.postgresql.org/docs/18/unsupported-features-sql-standard.html)

The proxy is not evidence of the actual Annex F text and must be replaced by a direct standards check before editor-ready submission.

## Provisional rules affected by this gate

- Store-assignment clause reference.
- Contextual typing.
- Privilege/access-rule wording.
- LOB/XML data-type restriction and T4xx-01 split.
- `INCLUDE` keyword treatment and clause placement.
- Assignment-target qualification.
- Omitted INSERT column-list terminology.
- Collation treatment.

The target is to reuse existing SQL semantic machinery wherever possible rather than introduce parallel rules.

## Proposal traceability matrix

| Proposal element | SQL:2023 material to verify | Decision required |
| --- | --- | --- |
| Data change delta table attachment | `<data change delta table>` and contained statement productions | Statement contexts and result-option applicability |
| INSERT clause placement | `<insert statement>` and `<override clause>` | Whether `INCLUDE` precedes or follows override syntax |
| Omitted insert list | INSERT Syntax Rules | Exact terminology and hidden/generated-column treatment |
| UPDATE target and correlation syntax | `<update statement>` and target/correlation productions | Whether ambiguity exists and where the clause fits |
| UPDATE assignment target | `<set clause>` and assignment-target productions | Whether unqualified include targets require a new rule |
| MERGE target and correlation syntax | `<merge statement>` and target productions | Proper attachment point and ambiguity behavior |
| MERGE insert specification | Column-list and value-source productions | Include-column membership and degree constraints |
| MERGE delete plus delta table | Existing MERGE and delta-table General Rules | Whether an exclusion is required or already implied |
| Include-column data types | Existing data-type applicability rules and feature dependencies | Whether the LOB/XML restriction and T4xx-01 boundary are supported by SQL, rather than only vendor precedent |
| Contextually typed values | Relevant value-specification Syntax Rules | Type context for `NULL`, parameter markers, and `DEFAULT` |
| Store assignment | Existing assignment General Rules | Exact cross-reference and conversion semantics |
| Access Rules | DML and query Access Rules | Whether “no additional privilege requirement” is sufficient |
| Character collation | Column/type and expression collation rules | Whether an explicit no-new-rule statement is correct |
| Keyword list | Reserved/non-reserved keyword provisions | Whether `INCLUDE` needs lexical or grammar treatment |
| Corrections and amendments | Corrigendum and in-ballot changes | Whether any relevant production has changed |
