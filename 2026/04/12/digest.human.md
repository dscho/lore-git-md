# Git Mailing List Digest — 2026/04/12 (Sunday)

**The day in brief.** A quiet Sunday with 14 emails across 7 threads, dominated by final approvals and documentation refinements. The standout development is Jeff King's final technical approval of Harald Nordgren's `git checkout -m` autostash series, clearing the last remaining issue (refname validation) before merging. Other notable activity includes final GSoC proposal submissions and continued polish on promisor file handling.

## Notable threads

### `git checkout -m` autostash series approved

Harald Nordgren's ten-iteration series enhancing `git checkout -m` with autostash behavior has received final approval from Jeff King, resolving the last outstanding issue around refname validation. The series now uses `CHECKOUT_AUTOSTASH_HEAD` as the refname, satisfying Git's rules for root-level refs. This concludes a comprehensive review process involving Junio Hamano, Phillip Wood, and Jeff King. The changes improve branch switching by automatically stashing uncommitted changes during conflicts, add customizable conflict marker labels to `git stash apply`, and implement silent autostash creation. With all technical concerns addressed, the series is ready for merging.

### GSoC proposal finalization

Pablo Sabater submitted the final version of his GSoC proposal to complete and extend Eric Ju's stalled work on `git cat-file --batch-command` remote object metadata support. The v4 proposal demonstrates Pablo's growing engagement with the community through merged test patches and active participation in reviews. Key technical elements include replacing unsafe `strstr()` validation with Jeff King's allow_list approach, fixing a cleanup bug between local/remote queries, and extending support for `%(objecttype)` through protocol changes. With mentorship from Christian Couder and positive review from Karthik Nayak, this appears a strong candidate for selection.

### Promisor file handling review

Junio Hamano provided final review feedback on Lorenzo Pegorari's promisor file repacking series, focusing on memory handling and input validation in patch 2/6. The review identifies several key issues needing attention: input validation for OIDs, NULL checks for `parse_pack_index()` returns, memory leak cleanup, and error path completeness. While maintaining the core approach is sound, Junio emphasizes defensive programming practices for this helper function. The feedback represents the last round of review before the series can be merged, addressing both correctness and potential optimizations.

## In brief

**`git stash` argument parsing** — Deveshi Dwivedi's final patch simplifies `git stash` to automatically use "push" when the command starts with any option flag, reducing verbosity for common workflows. Junio Hamano approved the design after four iterations of review.

**Glossary formatting fixes** — Jeff King and Kristoffer Haugsbakk continued discussing AsciiDoc formatting solutions for the Git glossary, refining terminology in commit messages while maintaining the technical approach combining blank lines and "--" delimiters.

**`git backfill` progress reporting** — Trieu Huynh proposed adding standard progress reporting to the experimental `git backfill` command, with Derrick Stolee raising important questions about interaction with underlying `git fetch` progress indicators.

**New `git recall` proposal** — Ammaar Bakshi RFC'd a new `git recall` command for formatted commit activity summaries, drawing parallels to how `git switch` simplified `git checkout`. The standalone implementation exists but questions remain about core Git scope.