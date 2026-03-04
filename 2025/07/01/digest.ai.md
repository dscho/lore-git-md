# Git Digest: 2025/07/01

**The day in brief.** A high-volume day with 151 emails across 31 threads, dominated by major architectural work (Patrick Steinhardt's `the_repository` removal series completion) and significant discussions around SHA-256 transition, reftable adoption, and clang-format policies. Notable progress on multiple fronts with several series reaching finalization.

## Notable threads

**ODB refactoring completes** -- Patrick Steinhardt's 17-patch series eliminating `the_repository` from Git's object database (ODB) subsystem reaches its final form (v6) with all review feedback addressed. The systematic conversion establishes explicit `object_database` parameters and consistent `odb_`-prefixed APIs while maintaining backward compatibility. Key changes include renaming core structures (`raw_object_store` → `object_database`), adding parent pointers for repository tracking, and converting all major ODB functions to explicit parameters. The series now provides a clean foundation for future pluggable backends. Justin Tobler's final review ack confirms readiness for merging after multiple iterations refining naming, comments, and path handling.

**SHA-256 default transition advances** -- brian m. carlson's 11-patch series implementing SHA-256 as the default hash algorithm (when built with WITH_BREAKING_CHANGES) reaches v2 with improved constant naming (`GIT_HASH_SHA1_LEGACY` replacing `GIT_HASH_ORIGINAL`). The changes introduce comprehensive test infrastructure (`GIT_TEST_BUILTIN_HASH`) and careful handling of legacy cases while preparing for Git 3.0. One open debate remains about whether commands outside repositories should use `GIT_HASH_DEFAULT` (author's preference) or `GIT_HASH_SHA1_LEGACY` (Patrick Steinhardt's suggestion for script compatibility). Junio Hamano approves the technical approach while noting outstanding documentation updates.

**Reftable adoption debate intensifies** -- Following discussions about case-insensitive ref conflicts, Patrick Steinhardt strongly advocates making reftable the default backend, citing its technical advantages and successful large-scale use at GitLab. Junio Hamano expresses surprise it wasn't already listed in Git 3.0's breaking changes and endorses using `features.experimental` as the opt-in mechanism. The discussion shifts to migration challenges, with Junio suggesting documentation to help script authors transition away from direct filesystem/refs/ access. This positions reftable as the long-term solution to case-sensitivity issues while acknowledging ecosystem adoption hurdles.

**Clang-format philosophy debate** -- The ongoing `.clang-format` discussion sees contrasting views emerge: Toon Claes and Johannes Schindelin advocate for full automation adoption (à la `gofmt`), while Junio Hamano demonstrates concrete cases where automated wrapping harms readability (in the `last-modified` series). Junio provides a patch correcting specific formatting regressions, arguing for human judgment where automation degrades code organization. The thread shows the tension between consistency and readability as Git considers broader formatting automation.

**In brief**

**`git last-modified` interface finalization** -- Kristoffer Haugsbakk and Junio Hamano settle on path handling behavior matching `git ls-tree` conventions (non-recursive by default, with `-r` for recursion). Junio identifies one final gap - missing help text implementation (`-h` flag) - before the feature can graduate from experimental status.

**Interactive patch diff context** -- Phillip Wood and Junio Hamano discuss final design questions in Leon Michalak's series, focusing on whether specialized option macros for `-U/--unified` are justified given their narrow use case. The core functionality is complete with positive reviews.

**Memory leak fixes** -- Lidong Yan's bitmap loading leak series (v6) and Jacob Keller's `send-pack` `extra_have` fix both finalize with comprehensive solutions. Junio notes one remaining test coverage gap for alternate-store push scenarios in the `send-pack` case.

**Windows credential manager workaround** -- Johannes Schindelin and brian m. carlson confirm the solution for suppressing Git Credential Manager popups: set an empty credential helper (`git config --global --append credential.helper ""`) before the desired helper. This persists across Git for Windows updates.

**On the radar**

**AI contribution policy** -- Initial proposal to forbid AI-generated content (similar to QEMU) sparks discussion about scope. Christian Couder notes the current wording may be overly broad compared to QEMU's code-specific prohibition. Junio agrees on distinguishing generative vs. assistive uses and invites refined language.

**Partial clone corruption report** -- Дилян Палаузов documents consistent failures in treeless/blobless clones where the first `git pull` after 5 days fails with missing object errors, while the second succeeds. This suggests promisor remote/commit-graph interaction issues that warrant investigation.