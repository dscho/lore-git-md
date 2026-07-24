## The day in brief

23 July 2026 was a busy day on the Git mailing list, with 46 emails across 20 threads. The standout developments: Junio Hamano’s procedural nudges on stalled documentation and completion topics, a **v3 bugfix series for `git mv`** that finally resolved its symlink-handling debate, and **Jeff King’s deep-dive critique** of the ODB performance regression, reframing it as a systemic alternates issue. Two new topics—**`http.connectTimeoutMS`** and a **`git clone --revision` segfault fix**—entered review, while Patrick Steinhardt’s **`remote-object-info`** series was effectively approved.

---

### Notable threads

### `git mv`’s symlink saga reaches resolution

Lucas Zamboni Orioli’s two-patch series fixing misleading errors in `git mv` when the destination directory is missing culminated in a **v3 iteration** that settled the symlink-handling debate. Junio C Hamano rejected the `stat()`-based approach (which follows symlinks) as unsafe for Git working trees, citing `git apply` as precedent for avoiding symlink traversal. The final implementation uses `lstat()` to check destination paths, explicitly rejecting symlinks to files while accepting symlinks to directories. The series now handles edge cases like non-directory intermediate components and includes thorough test coverage for `--dry-run` detection. With all substantive feedback addressed, the series is poised for integration into `next`.

### ODB performance regression: Peff broadens the scope

Jeff King (Peff) delivered a **substantive technical critique** of the ODB performance regression introduced by Patrick Steinhardt’s rework, reframing it as a **systemic alternates issue** affecting nearly every Git operation in large-scale hosting environments. His analysis revealed that the regression—previously thought to be quarantine-specific—impacts all uses of `GIT_ALTERNATE_OBJECT_DIRECTORIES`, including `clone -s` and `--reference`. Peff also challenged Patrick’s proposed transaction-aware ODB writes, arguing that cross-process dependencies (e.g., `index-pack`/`rev-list` coordination) would force the solution to replicate existing "alternates juggling," undermining its elegance. The discussion now splits into two tracks: a near-term lookup-order fix to restore performance, and Patrick’s long-term architectural redesign, delayed until late July or August.

### `git replay` documentation stalls, again

Junio Hamano nudged Kristoffer Haugsbakk to post a **v4 iteration** of his `git replay` configuration documentation series, which has been stalled for over a month despite being technically complete. The series synchronizes `replay.refAction` documentation between man pages using conditional linking and description lists, but Junio’s latest email suggests it may be discarded from the "What’s cooking" report if no update is sent soon. The blocker is purely procedural—no technical concerns remain.

### `show-branch` commit-slab refactoring: communication expectations set

Junio Hamano paused review of Gatla Vishweshwar Reddy’s **10th iteration** of the `show-branch` commit-slab refactoring to set clear expectations for mailing list communication. His email stressed that the list is a human collaboration space, discouraging rapid-fire patch iterations before rough consensus is reached. Gatla’s follow-up demonstrated engagement by explaining the root cause of a prior regression (an accidental removal of `init_commit_name_slab`) and committing to discuss questions before sending new versions. The thread is now technically ready for integration, pending Junio’s next move.

### `git repack --drop-filtered` RFC: safety guards elevated to blocking

Siddharth Asthana (a reviewer) raised the stakes for Siddharth Shrimali’s RFC series introducing `git repack --drop-filtered` for partial clones. He **elevated two safety guards**—merge/rebase/cherry-pick checks and index validation—to blocking prerequisites for the next RFC round, arguing they are non-negotiable to avoid immediate re-fetching of just-dropped blobs. The series’ **blocking logical flaw** (bitmap validation) remains confirmed, and the drop log’s documentation and test coverage are still deferred. The thread is in active design iteration, with no integration branch assigned yet.

---

### In brief

**`git bisect --reset-when-found`** -- Johannes Sixt provided **surface-level feedback** on the second patch of Harald Nordgren’s series, suggesting minor improvements to error message boilerplate, shell safety in tests, and test cleanup chaining. The series remains on track for `next`.

**Swift userdiff driver** -- Johannes Sixt **acked the v2 revision**, closing the review loop. The patch adds a built-in userdiff driver for Swift, replacing generic heuristics with language-aware patterns for function declarations and word boundaries. It is now fully reviewed and ready for integration testing.

**Documentation synopsis-style conversion** -- Jean-Noël Avila posted a **v2 reroll** of his series converting `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style. The reroll addresses Junio’s feedback on backtick usage and dash count in delimited blocks. The series is uncontroversial and likely to graduate to `next`.

**Submodule remote matching with `insteadOf`** -- Éric NICOLAS posted a **v2 bugfix** for `git submodule update --remote` when URL aliases are used. The patch resolves a regression introduced in 2025 by making remote matching respect `url.*.insteadOf` aliases. Junio’s feedback on commit message clarity and code simplification has been addressed.

**`git-p4` historical ambiguity** -- Hardik Kumar asked Junio to clarify or remove a **2007 `# fixme` comment** in `contrib/git-p4.py`, as the thread’s core question (whether the comment intended to exclude metadata, the subject, or both) remains unresolved. The thread has stalled, with no substantive technical feedback or test coverage.

**`http.connectTimeoutMS`** -- GalaxySnail introduced a new configuration option to control HTTP connection timeouts at millisecond granularity. Junio requested **use case justification**, clarification of zero-value behavior, and a decision on whether a dedicated test script is necessary.

**`git clone --revision` segfault fix** -- Adrian Friedli submitted a **one-line NULL check** to fix a segfault when using `--revision` with servers that advertise more refs than requested. Junio conceptually approved the fix but requested a **regression test** before merging.

---

### On the radar

**`ps/cat-file-remote-object-info`** -- Pablo Sabater reported that Karthik Nayak reviewed Patrick Steinhardt’s 13-patch series and found it ready, with only minor nits that do not warrant a reroll. The series adds a `remote-object-info` batch-command to `git cat-file` that fetches object metadata (e.g., size) from a remote server without downloading the object itself. It is now effectively approved and can proceed unless new issues surface.