## The day in brief

**2026/07/03** was a busy day on the Git mailing list, with **106 emails across 26 threads**. The traffic was a mix of **bugfixes, feature patches, test modernizations, and CI improvements**, with several long-running series reaching resolution. The most notable developments:

- **`git history drop`** (Patrick Steinhardt) was **merged** after resolving a critical ref-resolution dispute.
- **`git refs` subcommands** (`create`, `delete`, `update`, `rename`) were **merged** with minor follow-up discussion about symref ergonomics.
- **Reftable security hardening** (Patrick Steinhardt) reached **v3**, addressing all prior feedback and adding fuzzing infrastructure.
- **`GIT_TEST_LONG` CI improvements** (Patrick Steinhardt) landed in **v2**, making expensive tests more reliable and efficient.
- **`git replay --linearize`** (Toon Claes) saw **v6**, fully addressing post-merge regressions and documentation gaps.
- **`git rm` pathspec behavior** was clarified as **expected** (not a bug), with a focus on improving documentation.
- **`git repo info`** (GSoC) received **design feedback** about glob support for prefix queries.

The day’s tone was **collaborative and milestone-heavy**, with several significant features and fixes crossing the finish line.

---

## Notable threads

### `git history drop` merged after critical fix
**Subject:** [PATCH v8 0/11] Add `git history drop` subcommand
**Author:** Patrick Steinhardt

The long-running series adding `git history drop`—a command to remove a commit and replay its descendants—was **merged** after resolving a high-impact dispute over ref resolution in `find_head_tree_change()`. The v8 iteration addressed Junio’s concern by adding `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()`, ensuring correct identification of HEAD movements even in detached-HEAD states. The series also modernized the reset API, advanced `the_repository` removal, and added extensive test coverage for edge cases like bare repositories and conflict detection.

**Key takeaway:** The feature is now **ready for the next release**, expanding Git’s history-editing toolkit alongside `rebase` and `commit --amend`.

---

### `git refs` subcommands merged, symref ergonomics debated
**Subject:** [PATCH v3 0/5] refs: add delete, update, create, and rename subcommands
**Author:** Patrick Steinhardt

The series consolidating reference manipulation under `git refs` (adding `create`, `delete`, `update`, and `rename` subcommands) was **merged**, but follow-up discussion revealed usability concerns about symref handling. Toon Claes questioned whether `--no-deref` is intuitive for creating symbolic references, noting that the option name implies operating on the reference itself rather than its target. The debate highlights a tension between consistency with `git update-ref` and user expectations.

**Key takeaway:** The subcommands are **merged**, but future iterations may refine symref ergonomics.

---

### Reftable security hardening reaches v3
**Subject:** [PATCH v3 0/12] reftable: security hardening against corrupted files
**Author:** Patrick Steinhardt

The v3 iteration of the reftable security-hardening series **addressed all prior feedback**, including Toon Claes’s test robustness suggestions and Junio’s acknowledgment of the test helper improvement. The series now includes fuzzing infrastructure (libFuzzer + Meson support) and fixes for out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. The fuzzer has run for 2+ hours without surfacing new issues, indicating the hardening is comprehensive.

**Key takeaway:** The series is **technically complete** and ready for substantive review of the security fixes.

---

### `GIT_TEST_LONG` CI improvements land in v2
**Subject:** [PATCH v2 0/9] ci & test: make GIT_TEST_LONG tests reliable and efficient
**Author:** Patrick Steinhardt

The v2 series making `GIT_TEST_LONG` tests reliable and efficient for CI **landed**, addressing feedback from SZEDER Gábor and Jeff King. Key changes:
- Restored the `EXPENSIVE` prerequisite for `t4141-apply-too-large.sh` (memory usage >1 GiB).
- Added GitLab CI support for `GIT_TEST_LONG` on integration branches.
- Disabled the macOS RAM disk to avoid out-of-memory issues.
- Skipped broken tests on unsupported platforms (e.g., 32-bit systems).

**Key takeaway:** Expensive tests are now **more predictable and CI-friendly**, with better resource management.

---

### `git replay --linearize` v6 addresses post-merge regressions
**Subject:** [PATCH v6 0/3] replay: fix --linearize regression and clarify design
**Author:** Toon Claes

The v6 iteration of the `--linearize` option for `git replay` **fully addressed post-merge regressions** from v5, including silent commit dropping and merge commit divergence. The series now documents the design intent (producing a single linear sequence, even if it duplicates shared history) and adds test coverage for edge cases like divergent merges. Junio’s review raised a minor question about authorship attribution but did not block progress.

**Key takeaway:** The feature is **ready for re-merge**, replacing the flawed v5.

---

### `git rm` pathspec behavior clarified as expected
**Subject:** `git rm -n *.json` unexpectedly recurses into subdirectories
**Author:** Евгений Плискин

The thread clarified that `git rm -n *.json`’s recursive behavior is **expected**, not a bug. The issue stems from Git’s pathspec logic, where `*` matches directory separators by default. The discussion converged on improving documentation to explain the `:(glob)` pathspec modifier, which restricts matching to the current directory. No code changes are needed, but the man page for `git rm` may be updated to cross-reference `gitglossary(7)`.

**Key takeaway:** The behavior is **correct**, but documentation could be clearer.

---

## In brief

- **`git rev-list --exclude-first-parent-only` bugfix** -- Junio C Hamano fixed a misbehavior when additional commits are explicitly specified, adding a test case to `t/t6012-rev-list-simplify.sh`.
- **`git rm` recursion clarified** -- The recursive behavior of `git rm -n *.json` is expected due to pathspec rules; documentation may be improved.
- **`greplint.pl` v3 merged** -- Michael Montalbo’s test linter for converting `grep` to `test_grep` landed, addressing all prior feedback.
- **`git repo info` prefix queries** -- Junio suggested glob support (e.g., `layout.*`) for the GSoC feature, calling for a more flexible design.
- **`git history` signing support** -- Souma added GPG signing for `fixup`, `reword`, and `split` subcommands, with test coverage for config and flag overrides.
- **`git diff --index` proposal** -- A patch to replace `--cached` with `--index` received surface-level support but no substantive review.
- **`USE_NSEC` discussion** -- The debate about flipping the default for nanosecond timestamp support continues, with no consensus on runtime configuration or auto-detection.
- **Memory-leak fixes** -- Jeff King’s series plugging leaks in non-default hash implementations saw naming feedback but no technical objections.
- **macOS Unicode filename crash** -- A fix for `precompose_utf8` landed, addressing a crash with long UTF-8 filenames; test adjustments are pending.

---

## On the radar

- **`includeIf.worktree` symlink behavior** -- Patrick Steinhardt identified an inconsistency with `gitdir`; the author may need to align the two conditions.
- **`git replay --linearize` authorship** -- Junio questioned whether Toon Claes should take over authorship for the v6 iteration.
- **`GIT_TEST_LONG` policy** -- The unresolved question of how Git defines "expensive" tests (runtime vs. resource usage) may resurface in future discussions.