Here’s the digest for **2026/07/03**, covering the Git mailing list’s daily traffic:

---

# The day in brief
**July 3, 2026** was a **heavy but focused** day on the Git mailing list, with **106 emails across 26 threads**. The standout themes were **security hardening** (reftable backend fixes), **test/CI reliability** (expensive test optimizations), and **feature refinements** (worktree config, commit signing). Two threads dominated: Patrick Steinhardt’s **12-patch reftable security series** (now at v3) and his **9-patch CI/test efficiency series** (v2), both nearing completion. A **new bugfix** for `--exclude-first-parent-only` emerged, and **Junio’s "What’s cooking"** report clarified the status of several in-flight topics.

---

## Notable threads

### Reftable security hardening (v3)
**Subject**: [PATCH v3 0/12] reftable: security hardening against corrupted files
**Author**: Patrick Steinhardt
**Status**: **Ready for merging** after addressing all feedback.

Patrick’s **12-patch series** systematically hardens Git’s reftable backend against maliciously corrupted files, fixing vulnerabilities like out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The v3 update added a stricter test for patch 7/12 (heap-buffer-overflow fix) and introduced **fuzzing infrastructure** (libFuzzer + Meson support) to prevent regressions. Junio and Christian Couder approved the test helper in patch 5/12, and Toon Claes’s substantive reviews on patches 6–7 were resolved. The series is now **technically complete**, with no open questions, and represents a significant step forward for the reftable backend’s robustness.

**Why it matters**: Reftable files aren’t stored remotely, but local corruption could still trigger crashes or undefined behavior. The fuzzing integration is a reusable framework for future hardening work.

---

### CI/test efficiency (v2)
**Subject**: [PATCH v2 0/9] ci & test: make GIT_TEST_LONG tests reliable and efficient
**Author**: Patrick Steinhardt
**Status**: **Ready for merging** after addressing feedback.

This **9-patch series** tackles two problems: broken `GIT_TEST_LONG` tests in CI and GitLab’s lack of coverage for expensive tests. Key changes include:
- **Test correctness fixes**: Skipping `t0021` and `t7508` on platforms where they hang (64-bit and 32-bit, respectively).
- **Efficiency improvements**: Reducing peak disk usage in `t5608` and `t7900` by deleting repositories immediately after use.
- **CI alignment**: Enabling `GIT_TEST_LONG` in GitLab CI for integration branches (excluding Windows due to RAM limits).

The v2 update restored the `EXPENSIVE` prerequisite in `t4141` after SZEDER Gábor noted its high memory usage (>1 GiB), despite the runtime now being negligible. Jeff King (Peff) endorsed the disk-usage improvements, and Junio flagged a minor commit message nit in patch 2/9. The series is now **unblocked**, with all substantive feedback addressed.

**Why it matters**: Expensive tests are critical for catching edge cases, but they must be reliable and efficient to run in CI. This series ensures they are both.

---

### `--exclude-first-parent-only` bugfix
**Subject**: [PATCH] revision: fix `--exclude-first-parent-only` with explicit commits
**Author**: Junio C Hamano
**Status**: **New patch**, likely to merge.

Junio submitted a **standalone fix** for a bug in `git rev-list --exclude-first-parent-only` where explicitly specified commits (e.g., `git rev-list --exclude-first-parent-only F R1 ^R2`) were incorrectly excluded. The patch adds an early exit in `process_parents()` when the option is active and a parent is already marked as `SEEN`, preventing the exclusion logic from misfiring. A new test case in `t6012-rev-list-simplify.sh` verifies the fix.

**Why it matters**: This is a **regression fix** for a rarely used but documented option, ensuring it behaves as expected when combined with explicit commit ranges.

---

### Worktree config conditions (v6)
**Subject**: [PATCH v6 0/6] config: add `includeIf.worktree:<pattern>` support
**Author**: Chen Linxuan
**Status**: **Ready for merging** after addressing symlink behavior.

Chen’s **6-patch series** adds `includeIf.worktree:<pattern>` and `includeIf.worktree/i:<pattern>` to Git’s config system, allowing conditional inclusion based on the worktree’s realpath. The v6 update addressed Windows CI issues and improved documentation, but Patrick Steinhardt identified a **behavioral inconsistency**: unlike `gitdir`, the new `worktree` condition resolves symlinks to the real path, which could confuse users. The series is otherwise **technically sound**, with Junio approving the documentation and tests.

**Why it matters**: This feature simplifies multi-worktree setups by letting users scope configurations to specific directories (e.g., personal vs. work projects) without knowing Git’s internal `.git` structure.

---

### `git history` commit signing
**Subject**: [PATCH 0/3] history: teach `git history` to sign rewritten commits
**Author**: Souma
**Status**: **New series**, awaiting review.

Souma’s **3-patch series** teaches `git history`’s `fixup`, `reword`, and `split` subcommands to respect `commit.gpgsign` and the `-S/--gpg-sign` flags. The implementation plumbs signing options through the commit-creation logic in `replay.c` and adds regression tests for config-driven signing, command-line overrides, and descendant-commit replay. The series is **well-scoped** and aligns with Git’s existing signing infrastructure, but it hasn’t yet attracted review.

**Why it matters**: This extends Git’s signing capabilities to the experimental `git history` command, improving security for users rewriting history.

---

## In brief
- **`git rm` pathspec clarification**: Patrick Steinhardt and Phillip Wood clarified that `git rm -n *.json` recurses into subdirectories because Git treats arguments as **pathspecs** (not shell globs). The `:(glob)` modifier can restrict matching to the current directory.
- **`USE_NSEC` debate**: D. Ben Knoble and Jeff King confirmed that modern Linux filesystems (ext4, XFS, vfat) preserve nanosecond timestamps, but the discussion about whether to flip the default or make it runtime-configurable remains unresolved.
- **`git replay --linearize`**: Junio questioned the authorship of the v6 documentation patch but didn’t block the series, which is otherwise ready for re-merge after fixing a regression in v5.
- **Memory-leak fixes**: Jeff King’s **9-patch series** for non-default hash implementations (OpenSSL, libgcrypt) attracted naming feedback from Patrick Steinhardt, who suggested `git_hash_release()` over `git_hash_discard()`. The series is otherwise uncontroversial.
- **Test modernization**: Marcelo Machado Lage’s patch to modernize `t9811-git-p4-label-import.sh` received stylistic feedback from Patrick and Junio, who suggested splitting long commands and clarifying the commit message.
- **macOS Unicode crash**: Ihar Hrachyshka’s patch to fix a crash in `precompose_utf8` when handling long UTF-8 filenames (>255 bytes) was reviewed by Torsten Bögershausen and Patrick Steinhardt. Junio suggested rewriting the test to avoid Perl, and Patrick proposed using `FLEX_ALLOC_MEM()` for better memory management.
- **`git repo info` prefix querying**: K Jayatheerth’s GSoC patch to add category-based prefix querying to `git repo info` was critiqued by Junio, who suggested supporting glob patterns (e.g., `layout.*`) for greater flexibility.

---

## On the radar
- **`git rebase --edit-commits`**: Matthias Beyer’s feature request for programmatic commit editing during rebase was resolved by demonstrating `GIT_SEQUENCE_EDITOR`, but the discussion highlighted the complexity of shell-nesting in automation scripts.
- **`git diff --index`**: A proposal to deprecate `--cached` in favor of `--index` received vague support but no substantive review. The change would modernize Git’s CLI but risks breaking scripts.
- **`paint_down_to_common()` optimization**: Tian Yuchen’s **10-patch series** (v6) to optimize merge-base queries for one-sided histories is ready for merging after a procedural rebase. The series includes **100–1000x speedups** for asymmetric queries.