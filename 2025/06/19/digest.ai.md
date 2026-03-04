# Git Mailing List Digest - 2025/06/19

**The day in brief.** A moderately busy Thursday with 57 emails across 14 threads, featuring significant progress on performance optimizations, ongoing discussions about code formatting standards, and several bug reports. The standout items include Taylor Blau's finalized MIDX/cruft pack optimization series and a thought-provoking debate about `git checkout -b` behavior during rebase.

## Notable threads

### MIDX and cruft pack optimizations reach maturity

Taylor Blau's 9-part series optimizing MIDX and cruft pack interaction during repacking has reached its fifth iteration with all edge cases addressed. The series introduces configurable control over whether cruft packs should be included in MIDX files (`repack.midxMustContainCruft`), with production-validated performance improvements of 5-20% at GitHub. The final version handles missing objects during traversals and refines MIDX behavior for noop repacks, with all platform compatibility issues resolved by replacing non-portable `grep -o` with `sed` alternatives. The series has received thorough review from Elijah Newren and Junio Hamano, and appears ready for merging.

### Clang-format configuration debate intensifies

Junio Hamano sparked a wide-ranging discussion about Git's `.clang-format` configuration, questioning whether the tool's automated style changes actually improve code quality. The thread examined concrete examples where clang-format makes questionable formatting decisions, particularly around line wrapping and alignment in signature handling code. Christian Couder suggested configuration refinements to reduce false positives, while Brian m. carlson advocated for stricter enforcement similar to Go's `gofmt` philosophy. The discussion revealed fundamental tensions between mechanical consistency and human judgment in code formatting, with no clear resolution yet on how to improve the tool's integration into Git's workflow.

### Rebase behavior controversy

A bug report from Grant Birchmeier about `git checkout -b` succeeding during rebase (when regular `git checkout` would fail) evolved into a debate about whether this is a bug or a feature. Junio Hamano demonstrated how allowing branch creation during rebase enables useful workflows like splitting mixed commits into separate topic branches. This countered the original assumption that the behavior was dangerous and needed fixing. The thread highlights how Git's flexibility can lead to both power-user features and potential footguns, with the community now weighing whether to document this as intended behavior or modify it for safety.

## In brief

**Promisor-remote protocol enhancements** -- Christian Couder provided final approval for Karthik Nayak's series adding configurable validation of remote attributes, with all major design points settled after multiple iterations.

**Signature handling in fast-export/import** -- Elijah Newren raised substantive questions about the v4 patch's approach to representing commit signatures during conversion, suggesting the current implementation may be over-engineered for simple signature preservation.

**`git repo-info` GSoC project** -- Lucas Seiki Oshiro submitted v2 of the new repository metadata command, now with improved plaintext output format (key=value) and thorough test coverage for both files and reftable backends.

**`git fetch --prune` optimization** -- The performance improvement series gained consensus despite changing warning message ordering, with Jeff King noting the significant speed gains (410s to 4s) outweigh the minor output change.

**IMAP-send UX improvements** -- A straightforward patch made configuration error messages more helpful by including example `git config` commands to fix missing settings.

**Coccinelle checking enhancement** -- A build system change will now make `make coccicheck` fail when semantic patches suggest changes, rather than silently succeeding.

## On the radar

**Rustification effort** -- Ezekiel Newren's work to introduce Rust code into Git remains active but faces ongoing concerns from Randall S. Becker about platform support limitations.

**`the_repository` removal** -- Multiple GSoC participants (Ayush Chandekar and Lucas Seiki Oshiro) are working on refactoring to reduce global state, with `is_bare_repository()` specifically mentioned as a current target.