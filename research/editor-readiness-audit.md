# Editor-readiness audit — 2026-10-28

Status: **NOT EDITOR-READY**. The semantic proposal is substantial, but the current text is not a verified amendment to ISO/IEC 9075-2:2023 as corrected by Cor 1:2026.

## Verified corrections
- Proposal author corrected to Alaric Dailey.
- Optional-feature taxonomy references corrected from Annex F to Annex D (Annex F addresses incompatibilities).
- Committee-response description of LOB/XML restrictions aligned with the actual draft.

## Normative blockers

1. **Authoritative baseline and edit instructions.** Obtain authorized current SQL/Foundation working text, including ISO/IEC 9075-2:2023/Cor 1:2026, and provide exact clause/subclause-specific replacement/addition instructions, not a free-standing schematic. The August 2026 corrigendum is published: https://www.iso.org/standard/93691.html
2. **Access control / masking.** The existing privileges rule does not address whether an INCLUDE value can expose a target column covered by a column mask or row access policy. IBM Db2's data change table documentation has explicit rules for INCLUDE-derived values in the outermost SELECT. Determine whether existing SQL rules already cover this; otherwise specify non-bypass semantics. Do not assume “no extra privilege” means “no new access-control issue.” Source: https://www.ibm.com/docs/en/SSEPEK_12.0.0/pdf/db2z_12_sqlrefbook.pdf
3. **Grammar insertion.** Walk the actual <data change delta table>, INSERT, searched UPDATE, MERGE, assignment target, and relevant lexical productions. Show exact deltas for all three statements, including INSERT override/column list positions and INCLUDE keyword classification.
4. **MERGE semantics.** Verify applicability of MERGE in data change delta tables, operation-specific assignments, DELETE branch restrictions and OLD/NEW/FINAL combinations against the current baseline. Supply explicit rule wording for branch-local nullable results.
5. **Conformance.** Determine actual feature and subfeature dependencies, permissible data types, and implied-feature table changes. The LOB/XML split is currently a design choice, not a verified SQL:2023 constraint.
6. **Contextual typing and assignment.** Prove the type context of include-position NULL, DEFAULT and parameters, and the store-assignment rule, by exact cross-reference. Define how unused include columns acquire null values.
7. **Review result-option/view interactions.** For view targets, triggers, row images and row-level privileges, make the extra projected values consistent with existing delta-table semantics.

## Specific draft inconsistencies / test cases

- **INSERT degree vs DEFAULT VALUES:** Syntax Rule 5 imposes a degree test for a query expression / row value expression, while the example permits DEFAULT VALUES. State explicitly whether that rule is inapplicable to DEFAULT VALUES and why an include column then evaluates to null.
- **Non-updated / no-op rows:** “Only changed rows appear” needs a precise standard-based interpretation for matched UPDATE actions whose new values equal their old values, and for MERGE branches. Avoid accidentally requiring an actual value difference.
- **Duplicate and ambiguous names:** Confirm case-folding, delimited identifiers, target aliases, target view columns and implicit references under the SQL identifier and column-resolution rules.
- **MERGE source values:** Verify that a source expression evaluated into an include column produces exactly one corresponding value per affected target row; do not promise correlation for source rows that affect no target.
- **Security negative test:** A user allowed to UPDATE a masked column must not gain otherwise inaccessible original column values merely by copying it to an INCLUDE column.
- **Concurrency and failure:** Check aborted statements, triggers and cascades against existing atomicity and visibility rules. INCLUDE values must not outlive rolled-back data changes.
- **Result options:** Verify INSERT/UPDATE/MERGE OLD, NEW and FINAL applicability against normative text, not just Db2 examples.
- **Submission metadata:** Replace the document-number and feature-ID placeholders when assigned. Record the exact revision in submissions/ and cite the official disposition artifact when available.

## Minimal acceptance criteria

- [ ] Authorized normative SQL:2023 baseline plus Cor 1:2026 reviewed.
- [ ] Exact clause/paragraph-level edit script prepared.
- [ ] All provisional notes resolved or moved entirely to non-normative rationale.
- [ ] Access-control/masking rule justified or added.
- [ ] MERGE and result-option matrix verified.
- [ ] INSERT DEFAULT VALUES and contextual typing made unambiguous.
- [ ] Optional-feature taxonomy, dependencies and conformance text checked against Annex D and the feature-implication rules.
- [ ] Examples pass an explicit syntax/semantic test matrix; negative tests include collisions, omitted assignments and authorization bypass attempts.
- [ ] Document identity, revision, prior art and official FND-A20 disposition cited accurately.

This audit documents issues to resolve. It is not a claim that the current draft has been checked against the complete copyrighted normative text.
