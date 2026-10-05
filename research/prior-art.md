# Prior art and related mechanisms

This file separates supporting implementation research from the normative proposal in [`proposal.md`](../proposal.md). It records the evidence currently relied upon by the working draft; it is not itself normative.

## Db2 INCLUDE

| Behavior | Evidence status |
| --- | --- |
| INCLUDE on INSERT, UPDATE, DELETE, and MERGE | Demonstrated in Db2 documentation |
| Include columns are appended to the intermediate/result row | Demonstrated |
| Include columns are nullable; unassigned values are null | Demonstrated |
| Explicit DEFAULT for an include-column position yields null | Demonstrated in Db2 for z/OS MERGE documentation |
| Omitted include column in a MERGE insert list yields null | Demonstrated in Db2 for z/OS MERGE documentation |
| LONG VARCHAR, LONG VARGRAPHIC, LOB, XML, and derived distinct types are restricted in the documented z/OS MERGE form | Demonstrated |
| MERGE containing a DELETE action is rejected as a data-change table | Demonstrated; SQLCODE -270 |
| Include-only UPDATE/MERGE UPDATE and include-only MERGE INSERT assignment lists are rejected | Demonstrated; SQLCODE -20260, SQLSTATE 428G5 |
| INCLUDE combined with OLD TABLE | Design-derived from documented rules; no concrete example found |

Primary references:

- [Db2 for z/OS MERGE](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=statements-merge)
- [Db2 for z/OS INSERT](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=statements-insert)
- [Db2 for z/OS SQLCODE -20260](https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=codes-20260)

Db2's implicit insert column list covers every column not defined as implicitly hidden. That is useful prior art for the proposal's omitted-column-list case, but the SQL standard's exact terminology still needs to be verified.

## Related vendor mechanisms

- [PostgreSQL MERGE](https://www.postgresql.org/docs/18/sql-merge.html) in PostgreSQL 17+ provides `MERGE ... RETURNING`, which can reference source columns. PostgreSQL `INSERT ... RETURNING` cannot reference source columns from an `INSERT ... SELECT`.
- [SQL Server MERGE](https://learn.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-ver17) provides `OUTPUT`, which can reference source columns for MERGE. SQL Server `INSERT ... OUTPUT` cannot reference the source columns of an `INSERT ... SELECT` in the same way.
- [jOOQ data change delta tables](https://www.jooq.org/doc/latest/manual/sql-building/table-expressions/data-change-delta-tables/) independently documents the OLD/NEW/FINAL row-image model used by Db2-style delta tables.

These are related mechanisms, not evidence that the proposed syntax or semantics are already standardized.

## Gap being addressed

The portable gap is not merely returning generated values. It is preserving a source-side or operation-local correlation value on the same relational row as the target row image produced by the data change.

The asymmetry is significant: PostgreSQL and SQL Server both recognize the need to expose source-side values for MERGE, but not through ordinary INSERT source correlation. In practice, an INSERT-only use case can be forced through MERGE with an `ON 1 = 0` pattern, which is a vendor-specific workaround rather than a portable T495 mechanism. Vendor RETURNING/OUTPUT facilities therefore solve adjacent cases, but do not establish one portable mechanism for passing non-target values through a data change delta table.
