Here's the daily digest for February 6, 2026:

---

### The day in brief
February 6 saw active development across Git's core functionality, with 112 emails across 27 threads. The day was dominated by technical refinements to ongoing efforts like `the_repository` removal, sparse-index integration, and reference backend selection. Notable highlights include a security vulnerability report in `git am`'s patch parsing and progress on several major refactoring efforts nearing completion.

---

### Notable threads

**Security vulnerability in `git am` patch parsing**  
A serious security issue was reported where `git am` can inadvertently apply diffs embedded in commit messages, potentially introducing malicious changes. The thread revealed this was responsible for a real-world incident where a `sleep(1)` call was accidentally added to a Debian package. Discussion centered on parsing challenges and backward compatibility concerns, with Jeff King (Peff) providing historical context showing this is a long-standing known issue. While no immediate solution was proposed, the thread established consensus that this warrants a fix despite the technical challenges.

**Reference backend selection nears completion**  
Karthik Nayak's series enabling configurable reference storage locations (files<->reftable) received final review from Patrick Steinhardt, addressing remaining documentation nits. The implementation now supports both config-based (`extensions.refStorage`) and environment-variable-based (`GIT_REFERENCE_BACKEND`) control, with worktree integration fully resolved. Junio Hamano signaled approval, indicating this major infrastructure change is ready for integration after multiple iterations.

**Submodule `ignore=all` behavior finalized**  
Claus Schneider's series aligning `git add` behavior with `git status` for ignored submodules reached v4, now requiring `--force` to stage changes to submodules marked with `ignore=all`. The comprehensive solution includes new helper functions, thorough test coverage, and documentation updates across three man pages. The changes address a long-standing inconsistency in submodule handling while maintaining backward compatibility.

**Meson build system completes gitk/git-gui integration**  
Patrick Steinhardt confirmed Windows compatibility for the symlink-based Meson integration of Git's GUI components, resolving the last outstanding issue (a missing `msgfmt` dependency in CI). Junio outlined the final integration path via subtree updates from Johannes Sixt's gitk repository. This marks a significant milestone in the build system modernization effort.

**Interactive hunk selection gains inter-file navigation**  
Abraham Samuel Adekunle's series adding `--rework-with-file` to `add -p` and related commands progressed through review, with Junio providing style and control flow feedback. The feature allows navigating between files during interactive patch selection using '>' and '<' keys, addressing a workflow limitation where users were previously forced through files linearly.

---

### In brief

**Whitespace/symlink bugfix** -- Junio C Hamano finalized a fix for incorrect whitespace checking of symbolic links, with Patrick Steinhardt's review confirming the refined handling of text-to-symlink transitions.

**PID file debugging merged** -- Patrick Steinhardt gave final approval for the lockfile PID tracking feature after six iterations, noting only an optional test cleanup remains.

**Documentation standardization** -- Jean-Noël Avila's synopsis-style conversions for `git-submodule` and `git-show` man pages received Kristoffer Haugsbakk's Reviewed-by after multiple refinement rounds.

**ODB enumeration API** -- Junio noted Taylor Blau's requested discussion about mtime handling in Patrick Steinhardt's object database refactoring hasn't yet materialized.

**Config batch proposal** -- Derrick Stolee's RFC for batched config operations received documentation polish from Jean-Noël Avila, focusing on man page formatting standards.

**`merge-ours` sparse-index support** -- Sam Bostock's series adding sparse-index compatibility to the merge strategy reached v2 with improved commit messages addressing maintainer feedback.

**Const-correctness warnings** -- Collin Funk's series addressing glibc 2.43 warnings was approved pending minor commit message tweaks from Ben Knoble.

---

### On the radar

**Rust interoperability** -- brian m. carlson's work on SHA-1/SHA-256 interoperability continues, with Windows CI issues being the current blocker.

**Hook configuration** -- Adrian Ratiu's series enabling config-based hooks faces questions from Junio about the fundamental motivation for the feature.

**Graph simplification RFC** -- Toon Claes proposed a `--untangle` option for `log --graph` to simplify complex merge visualization, receiving positive initial feedback from Junio.

**AI-assisted l10n** -- Jiang Xin's controversial proposal for translation assistance advanced slightly with documentation refinements, though opposition remains strong.

--- 

The day's activity showed Git's development progressing on multiple fronts, with particular focus on security hardening, infrastructure modernization, and user experience improvements. Several major efforts appear poised for integration in the near future.