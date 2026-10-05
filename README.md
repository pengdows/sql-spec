# sql-spec

Working repository for SQL standardization proposals developed from implementation experience in the Pengdows ecosystem.

## Current proposal

**[Include Columns for Data Change Delta Tables](proposal.md)** extends SQL feature T495 (Combined data change and retrieval) with declared pass-through columns for INSERT, UPDATE, and MERGE.

The motivating problem is source-row correlation: a data change delta table returns target-row images, but values used to identify or correlate the source row are lost when they are not stored in the target.

The proposal is a work in progress. Core feature semantics are specified; exact integration with the current ISO/IEC 9075-2 working text and the disposition of P02-USA-200 remain research gates.

## Repository layout

- [proposal.md](proposal.md) — canonical working proposal
- [research/prior-art.md](research/prior-art.md) — vendor prior art and related mechanisms
- [research/P02-USA-200.md](research/P02-USA-200.md) — standards-history research gate
- [research/sql-standard-alignment.md](research/sql-standard-alignment.md) — SQL standard integration checklist
- [research/committee-response-posture.md](research/committee-response-posture.md) — likely committee questions and response posture
- [submissions/README.md](submissions/README.md) — immutable records of formal submissions

## Working model

`proposal.md` is the canonical development source. Formal committee submissions should be generated from a tagged repository revision so the exact submitted text remains reproducible.

The intended workflow is: tag the exact revision, generate the committee-facing artifact, record the submission and disposition under [`submissions/`](submissions/), and link articles directly to [`proposal.md`](proposal.md) rather than only to the repository root.

Committee-owned working drafts and substantial copyrighted standards text should not be committed to this repository.
