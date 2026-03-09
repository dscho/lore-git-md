# Git Mailing List Digest - 2025/12/01 -- 2025/12/07

**The week in brief.** A busy week with 326 emails across 95 threads, featuring significant progress on multiple fronts. Key developments include the completion of Julia Evans' Git data model documentation, major advancements in reference backend configuration, and security hardening around terminal escape sequences. The experimental `git-history` command saw extensive refinement, while Windows compatibility work and performance optimizations for promisor objects also stood out. Junio Hamano's "What's cooking" report provided a comprehensive snapshot of the project's current state.

## Key developments

### Git data model documentation finalized

Julia Evans' long-running effort to document Git's core data model reached completion this week with Patrick Steinhardt's final review approval. The seven-iteration series introduces `gitdatamodel.adoc`, explaining objects, references, the index, and reflogs in accessible yet technically precise language. This fills a critical gap in Git's official materials, with reviewers praising its balance of accuracy and clarity. The standardized terminology (notably "file type" over "file mode") reflects careful consensus-building across multiple iterations.

### Reference backend configuration consensus

Karthik Nayak's series enabling reference backend selection via URI syntax (`<backend>://<path>`) saw substantial discussion, culminating in consensus around Patrick Steinhardt's proposal to extend the `extension.refStorage` config. The v4 implementation will use `GIT_REFERENCE_BACKEND` (renamed from `GIT_REF_URI`) while treating storage locations as backend-specific opaque strings. Junio Hamano endorsed this approach as it better accommodates future needs than environment variables alone. The design decisions here have implications for the ongoing object database refactoring work.

### Terminal escape sequence security debate

A heated but technically grounded debate emerged about mitigating terminal escape sequence vulnerabilities in Git's sideband channel. Johannes Schindelin demonstrated concrete risks of sequences like `OSC P 1 0 ; ? ST`, while brian m. carlson showed SSH channel limitations. The philosophical divide centers on whether Git should attempt comprehensive sanitization or rely on terminal emulator hardening. While no final resolution was reached, Schindelin's proposal for a granular `sideband.allowControlCharacters` configuration gained traction as a potential compromise.

### git-history command development

Patrick Steinhardt's experimental `git-history` command saw extensive refinement across seven iterations this week. The v7 series introduces `reword` and `split` subcommands built on replay infrastructure, with significant refactoring of add-patch and cache-tree subsystems to support in-memory operations. While the current focus remains on single-branch operations (despite Elijah Newren's concerns about multi-branch commit handling), user demand for previously removed `drop` and `reorder` subcommands suggests future expansion directions. The series now has comprehensive test coverage and appears ready for experimental merging.

### Windows compatibility and performance work

Johannes Schindelin's 10-patch series preparing Git's test suite for MSYS2's upcoming symlink support reached v2 with maintainer approval. Meanwhile, Aaron Plattner and Jeff King collaborated on dramatic performance improvements for promisor pack handling - reducing processing time for a 176GB pack from 76 minutes to under 2 minutes by avoiding unnecessary blob parsing. Taylor Blau also landed a 17-patch MIDX compaction series that combines adjacent MIDX layers while preserving bitmap validity, a key component of Git's incremental repacking strategy.

## In brief

**Security hardening** -- René Scharfe completed a series eliminating insecure `mktemp(3)` usage, introducing `git_mkdtemp()` and banning the vulnerable function.

**Hook subsystem refactoring** -- Adrian Ratiu and Emily Shaffer's 11-part series converting Git's hook execution to a structured API reached final form, enabling future config-based hooks.

**Submodule path encoding** -- Junio Hamano provided decisive guidance, clarifying that repositories with the extension enabled must treat `submodule.gitdir` as authoritative.

**git replay improvements** -- The command gained `--revert` capability and saw documentation enhancements clarifying its silent conflict behavior.

**Scalar configuration** -- Derrick Stolee's comprehensive audit and documentation of Scalar's settings concluded, making its behavior more transparent.

**Build system fixes** -- Toon Claes addressed Meson build issues for s390x cross-compilation, particularly around ICONV_OMITS_BOM detection.

**Lockfile debugging** -- Paulo Casaretto introduced optional PID tracking for diagnosing lock conflicts, though naming convention debates remain unresolved.

**Git v2.48.0-rc1** -- Junio announced the first release candidate including documentation improvements and whitespace handling enhancements.

**Data loss bug** -- A concerning report demonstrated potential data loss during `git pull --rebase` with multiple push URLs, reproducible across versions.

## Looking ahead

The `git-history` command appears poised for experimental merging in the coming week, while the reference backend configuration work will likely see its URI syntax implementation. Security discussions around terminal handling may yield concrete configuration options, and the Windows symlink support work is nearing integration. The data loss bug during rebase with multiple push URLs warrants urgent investigation given its potential severity.