# The Git Project Mailing List Daily Digest

**The day in brief.**
July 3, 2026 (UTC) was a **heavy-traffic day** with 106 emails across 26 threads, dominated by **security hardening**, **test infrastructure improvements**, and **feature refinements**. The standout developments: Patrick Steinhardt’s **reftable security-hardening series (v3)** is now complete and ready for merging, while Jeff King’s **memory-leak fixes for non-default hash implementations** sparked a lively debate about API design. A **new `git history` signing feature** and **worktree-based config inclusion** also advanced, with the latter drawing substantive feedback about symlink handling.

---

## Notable threads

### Reftable security hardening (v3) -- Patrick Steinhardt
Patrick Steinhardt’s **12-patch series** systematically hardens Git’s reftable backend against maliciously corrupted files, addressing vulnerabilities like out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The v3 update incorporates feedback from Toon Claes, making test cases stricter by dynamically calculating corruption values instead of hardcoding them. The series also introduces **fuzzing infrastructure** (libFuzzer + Meson support) to prevent regressions, a notable addition to Git’s CI tooling. Junio C Hamano and Christian Couder have acknowledged the test helper improvements, and the series is now **technically complete and ready for merging**. The hardening is comprehensive, though exploitation requires local disk access, and the fuzzer has run for 2+ hours without surfacing new issues.

### Memory leaks in non-default hash implementations -- Jeff King
Jeff King’s **9-patch series** plugs memory leaks in Git’s hash context (`git_hash_ctx`) when using non-default backends (OpenSSL SHA-256, libgcrypt). The leaks, invisible with Git’s default hash backends, were discovered via `SANITIZE=leak` and affect subsystems like `csum-file`, `patch-id`, and HTTP object requests. Patch 1/9 (removing a redundant `discard_hashfile()` function) received maintainer approval, but the series sparked a **substantive debate** about API design. Patrick Steinhardt and Brian M. Carlson advocated for making `git_hash_discard()` **idempotent**, citing Rust integration benefits and historical pain with non-idempotent APIs (e.g., reference transactions). Peff’s current implementation uses a flag-based approach to avoid double-discarding, but the discussion highlights a tension between pragmatism and long-term API hygiene. The series is **ready for review**, with the idempotency question deferred to a potential follow-up.

### `GIT_TEST_LONG` tests: reliability and CI efficiency (v2) -- Patrick Steinhardt
Patrick Steinhardt’s **9-patch v2 series** makes `GIT_TEST_LONG` tests reliable and efficient enough to run in CI, addressing broken or hanging tests in GitHub and GitLab pipelines. The series splits into three categories: **CI visibility** (adding a GitLab badge, enabling `GIT_TEST_LONG` in GitLab CI), **test correctness fixes** (skipping tests on unsupported platforms), and **efficiency improvements** (reducing disk/CPU usage). Patch 3/9, which refactors `t4141-apply-too-large.sh` to replace a slow `dd` loop with `genzeros`, drew feedback from SZEDER Gábor about retaining the `EXPENSIVE` prerequisite due to high memory usage (>1 GiB). Patrick conceded the point, restoring the label in v2. Junio C Hamano flagged minor wording nits in two patches, but the series is otherwise **uncontroversial and ready for integration**. The changes are mechanical and well-motivated, with clear commit messages explaining each fix.

### `git history` signing support -- Souma
Souma’s **3-patch series** teaches `git history` to sign rewritten commits via the `fixup`, `reword`, and `split` subcommands. The patches respect the `commit.gpgsign` configuration and the `-S/--gpg-sign`/`--no-gpg-sign` command-line options, with thorough test coverage for configuration-driven signing, command-line overrides, and descendant-commit replay. The implementation extends the commit-creation logic in `builtin/history.c` and `replay.c` to pass signing options through to the underlying commit machinery. The series is **well-scoped and technically sound**, with no prior discussion or objections in the thread. It aligns with Git’s existing signing infrastructure and would be a useful addition for users who want to ensure the integrity of rewritten history.

### `includeIf.worktree:<pattern>` (v6) -- Chen Linxuan
Chen Linxuan’s **6-patch v6 series** introduces `includeIf.worktree:<pattern>` and `includeIf.worktree/i:<pattern>` conditions to Git’s config system, allowing conditional config inclusion based on the working tree’s realpath. The feature addresses a gap in multi-worktree setups, where `gitdir` patterns are cumbersome. The implementation reuses existing pattern-matching infrastructure and includes comprehensive test coverage for edge cases like symlinked paths and case sensitivity. Junio C Hamano endorsed the current approach but suggested consolidating documentation for the case-sensitive/insensitive variants in a follow-up. **Patrick Steinhardt raised a substantive issue**: the new `worktree` condition resolves symlinks to the real path, unlike `gitdir`, which matches both symlinked and real paths. This inconsistency could confuse users, and Patrick’s test case demonstrates the discrepancy. The series is otherwise **ready for merging**, but the symlink behavior should be resolved (either by aligning `worktree` with `gitdir` or documenting the divergence).

---

## In brief

**`git rev-list --exclude-first-parent-only` bugfix** -- Junio C Hamano fixed a misbehavior where the option incorrectly excluded commits when additional commits were explicitly specified. The patch adds a new test case and targets `process_parents()` in `revision.c`.

**`git rm -n *.json` recursion clarification** -- Patrick Steinhardt and Phillip Wood clarified that `git rm`’s recursive behavior with glob patterns is **expected** (not a bug) due to Git’s pathspec rules. The discussion highlighted the `:(glob)` pathspec modifier as a workaround for non-recursive matching.

**`git replay --linearize` (v6)** -- Toon Claes’s series addressing post-merge issues in `git replay --linearize` is now **ready for final review**. Junio C Hamano raised a minor question about authorship attribution but did not object to the technical content.

**`git refs` subcommand follow-ups** -- Toon Claes raised usability questions about `git refs create`’s `--no-deref` option and `git refs rename`’s symref support. The feedback is **surface-level** and does not challenge the merged series’ correctness.

**`git repo info` prefix querying** -- K Jayatheerth’s GSoC patch added category-based prefix querying to `git repo info`, but Junio C Hamano suggested using **globs** (e.g., `layout.*`) instead of simple prefixes for greater expressiveness.

**`git history drop` series merged** -- Patrick Steinhardt’s 11-patch series adding `git history drop` is now **fully merged** after Junio C Hamano addressed a whitespace nit. The series modernizes the reset API and advances `the_repository` removal.

**`greplint.pl` (v3)** -- Michael Montalbo’s **6-patch series** introducing `greplint.pl` (a linter to convert bare `grep` to `test_grep`) is now complete. The series addresses all prior feedback, including audits of `# lint-ok` exemptions and fixes for pre-existing test bugs.

**`git rm` pathspec documentation gap** -- The thread about `git rm -n *.json` unexpectedly recursing into subdirectories **converged on a documentation gap**. The behavior is correct (due to pathspec rules), but users may not find the `:(glob)` modifier in `git-rm(1)`. No patch has been proposed yet.

**macOS Unicode filename crash fix** -- Ihar Hrachyshka’s patch fixing a crash in Git’s `precompose_utf8` layer when handling long UTF-8 filenames received **substantive feedback** from Patrick Steinhardt and Junio C Hamano. The discussion focused on memory management and test robustness, with Junio suggesting rewriting the test to avoid Perl.

**`git rebase --edit-commits` feature request** -- Matthias Beyer’s request for a programmatic way to edit commits during interactive rebase was **resolved by existing features**: `GIT_SEQUENCE_EDITOR` and `git log --format=%h`. Brian M. Carlson and D. Ben Knoble provided practical examples and caveats.

---

## On the radar

**`paint_down_to_common()` optimization (v6)** -- Tian Yuchen’s series optimizing `paint_down_to_common()` for one-sided histories is **ready for merging** after a procedural rebase. Junio C Hamano has not yet queued it in `seen`.

**`USE_NSEC` discussion** -- The debate about whether to flip the default of `USE_NSEC` to `true`, deprecate the knob, or convert it to runtime configuration remains **unresolved**. Jeff King’s testing showed modern Linux filesystems preserve nanosecond timestamps, but interoperability risks (e.g., Git/JGit mixing) and platform coverage gaps persist.

**`git apply` quadratic-time behavior in reftable** -- Kristofer Karlsson’s patch fixing a quadratic-time scalability issue in the reftable backend is **stalled**. The fix exposes tombstones to iterator bounds checks, but Kristofer reports difficulty reproducing the performance improvement in realistic scenarios. The patch’s practical value is now in question.