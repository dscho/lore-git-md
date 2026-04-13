# Git Mailing List Digest — 2026/04/12 (Sunday)

**The day in brief.** A quiet Sunday with 14 emails across 7 threads, dominated by final approvals and documentation refinements. The standout development is Jeff King's final technical approval of Harald Nordgren's `git checkout -m` autostash series, clearing the last remaining issue (refname convention) for merging. Other notable activity includes a GSoC proposal finalization and progress on promisor file handling.

---

### Notable threads

**Autostash behavior for conflicted branch switches finalized**  
Harald Nordgren's long-running series to enhance `git checkout -m` with autostash behavior during conflicted branch switches has received final technical approval from Jeff King, resolving the last outstanding issue about refname conventions. The v10 patch renames the autostash ref to `CHECKOUT_AUTOSTASH_HEAD` to comply with Git's documented rules for root-level refs. The series now includes silent autostash creation and customizable conflict marker labels for `git stash apply`, with comprehensive test coverage across 19 test cases. Junio C Hamano had previously approved the series, and with this final naming confirmation, it appears ready for merging after 10 iterations of review.

**GSoC proposal to complete remote object metadata support**  
Pablo Sabater has finalized his GSoC proposal to complete and extend Eric Ju's stalled work on `git cat-file --batch-command` remote object metadata support. The v4 proposal demonstrates Pablo's growing engagement with the community through merged test patches and identifies a new bug in the original implementation where `data->type` isn't properly cleared between queries. The work will be split into two phases: first fixing the existing implementation (replacing unsafe `strstr()` validation with Jeff King's allow_list approach), then extending support for `%(objecttype)` through protocol changes. Christian Couder will mentor the project, with Karthik Nayak providing technical review.

**Promisor file handling during repacks**  
Junio C Hamano has provided detailed review feedback on Lorenzo Pegorari's patch series improving promisor file handling during repacks, focusing on memory safety and input validation in the `copy_promisor_content()` helper function. The review identifies several issues including missing input validation for OIDs, NULL check gaps, and memory leaks that need addressing before merging. The series, part of Lorenzo's GSoC work mentored by Eric Sunshine, has already resolved CI issues and now awaits these final fixes. The changes are localized to promisor file maintenance during geometric repacking scenarios.

---

### In brief

**`git stash` argument parsing improvements** — Deveshi Dwivedi's final version simplifies `git stash` to automatically assume "push" when any option flag is present, reducing command-line verbosity. The change is well-tested and approved by Junio C Hamano.

**Glossary formatting fixes** — Jeff King and Kristoffer Haugsbakk continue refining AsciiDoc formatting for the Git glossary, finalizing a hybrid solution that works across both asciidoc and asciidoctor toolchains.

**`git backfill` progress reporting** — Trieu Huynh proposes adding standard progress reporting to the experimental `git backfill` command, with Derrick Stolee raising important questions about interaction with underlying `git fetch` progress indicators.

**New `git recall` proposal** — Ammaar Bakshi RFCs a new `git recall` command for formatted commit activity summaries, drawing parallels to how `git switch` simplified `git checkout`. The implementation exists as a standalone tool.