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
- [ ] Verify datatype feature dependencies before freezing T4xx-01.
- [ ] Verify whether assignment-target qualification is already prohibited by the existing grammar.
- [ ] Check corrigenda, amendments, and current in-ballot/current-draft changes touching `<data change delta table>`.
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
- LOB/XML datatype restriction and T4xx-01 split.
- `INCLUDE` keyword/correlation-name ambiguity.
- Assignment-target qualification.
- Omitted INSERT column-list terminology.
- Collation treatment.

The target is to reuse existing SQL semantic machinery wherever possible rather than introduce parallel rules.
