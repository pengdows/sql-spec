# Committee response posture

This document is non-normative. It records the likely technical questions raised by a committee reader and the evidence-backed response the proposal should provide.

## Why this feature belongs in SQL

The motivating requirement is already recorded in P02-USA-200: a data change delta table should be able to pass additional information, including the MERGE branch taken, to the containing query. The comment cites the 2004 VLDB paper that introduced INCLUDE columns in an implemented SQL extension.

The proposal is therefore not standardizing a private vendor convenience. It supplies portable wording for a requirement already identified in the SQL standardization process and preserves the existing relational data-change-delta-table model.

## Likely questions and responses

| Likely question | Response posture |
| --- | --- |
| Why not use session state or a generated-key lookup? | Session state is ambiguous across statements, triggers, concurrent activity, and multi-row MERGE. A delta-table row keeps the correlation value with the target-row image produced by the same operation. |
| Why not promise output order or source-row ordinals? | SQL does not generally provide those guarantees. Correlation must be carried as a value, not inferred from row position or identity contiguity. |
| Why not standardize `RETURNING` instead? | T495 already supplies the standard relational result mechanism. T4xx adds a projection channel to that mechanism without introducing a second top-level DML-returning construct. |
| Why are include columns not target columns? | They are operation-local result columns. They must not alter storage, target constraints, generated columns, referential actions, transition tables, or triggered actions. |
| Why are include values appended to the row type? | Appending preserves the existing target image and gives every INSERT, UPDATE, and eligible MERGE branch one stable result shape. |
| Why are DELETE and MERGE delete branches initially excluded? | Standalone DELETE has no independent source relation to correlate, and a MERGE delete branch has no corresponding NEW target-row image. These are scope decisions that can be revisited without changing the core include-column semantics. |
| Why does the proposal not copy Db2's type restrictions? | Db2 restrictions are recorded as prior art only. The normative feature follows SQL's own data-type applicability rules; vendor behavior does not define the standard. |
| What remains unresolved? | Exact SQL:2023 grammar insertion points, keyword treatment, contextual typing cross-references, access rules, collation rules, MERGE-delete interaction, and any effect of Cor 1:2026 or current ballot text. |

## Alternatives considered

The proposal should state that it considered and rejected, for the initial feature:

1. Session-scoped generated-key state.
2. Output ordinals or implied row ordering.
3. A new top-level `RETURNING` construct.
4. An action-specific MERGE pseudo-column.
5. Vendor-specific type restrictions as normative SQL requirements.

These alternatives either weaken relational semantics, create portability problems, or duplicate existing T495 machinery.

## Submission evidence package

Before formal submission, preserve:

- The exact P02-USA-200 ballot text and the current disposition inquiry.
- The VLDB prior-art citation.
- The normative proposal revision and exact SQL:2023 edit instructions.
- A response to each identified technical objection.
- The document number, submission date, meeting, and eventual disposition.
