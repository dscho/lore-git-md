# The Git Project Mailing List Daily Digest

## The day in brief.

July 3, 2026, saw **106 emails across 26 threads**, a busy but focused day. The standout developments: Patrick Steinhardt’s **`git history drop`** series was **merged after a critical ref-resolution dispute was resolved**, and Jeff King’s **reftable security-hardening series** (12 patches) reached v3 with all feedback addressed. A **macOS Unicode filename crash** was fixed, and **`GIT_TEST_LONG` tests** were made CI-friendly in a nine-patch series. The day also featured **ongoing debates** about `USE_NSEC`’s future (build-time vs. runtime), a **new `git repo info` prefix query** from a GSoC contributor, and a **feature request for signed `git history` commits**.

---

## Notable threads

### `git history drop` merged after critical dispute resolution
**Patrick Steinhardt’s 11-patch series** adding the `git history drop` subcommand was **merged into `next`** after resolving a high-impact logical flaw in the ref-resolution logic. The series modernizes the reset API, advances `the_repository` removal, and includes **561 lines of test coverage** for edge cases like detached `HEAD` and bare repositories. Junio C Hamano applied a trivial whitespace fix locally, closing the thread. The command enables removing a commit and replaying its descendants, a long-requested feature for history editing.

**Key detail:** The dispute centered on `find_head_tree_change()`’s behavior with `--update-refs=head` in detached-HEAD states. The v8 fix added `RESOLVE_REF_READING` to ensure the function returns `NULL` for unresolved `HEAD`, preventing a mismatch between symbolic branch names and `HEAD` in the ref-update array.

---

### Reftable security hardening reaches v3
**Jeff King’s 12-patch series** hardening Git’s reftable backend against maliciously corrupted files reached **v3**, addressing all prior feedback. The series includes **fuzzing infrastructure** (libFuzzer + Meson support) and fixes for out-of-bounds reads/writes, `abort()` calls, and uninitialized memory usage. Toon Claes’s review led to a stricter test in patch 7/12, which now dynamically calculates corruption values to avoid off-by-one errors. The series is **technically complete** and ready for integration, with no open questions.

**Key detail:** The fuzzer, now integrated into Git’s CI via `ci/run-build-and-minimal-fuzzers.sh`, exercises the three main entry points into the reftable parser and has run for 2+ hours without surfacing new issues.

---

### `GIT_TEST_LONG` tests made CI-friendly
**Patrick Steinhardt’s nine-patch v2 series** made `GIT_TEST_LONG` tests reliable and efficient enough to run in CI. The series fixes broken tests (e.g., `t0021-conversion.sh` on 64-bit platforms), reduces disk usage (e.g., `t7900-maintenance.sh` now isolates its 2 GB repository), and enables `GIT_TEST_LONG` in GitLab CI for integration branch pushes. Junio C Hamano noted a minor commit message nit in patch 2/9, but the series is otherwise **ready for merging**.

**Key detail:** Patch 3/9 restores the `EXPENSIVE` prerequisite for `t4141-apply-too-large.sh` after SZEDER Gábor argued that memory usage (>1 GiB) still justifies the label, even though runtime was reduced to under a second.

---

### macOS Unicode filename crash fixed
**Ihar Hrachyshka’s patch** fixed a crash in Git’s `precompose_utf8` layer when handling UTF-8 filenames longer than `NAME_MAX` (255 bytes) on macOS. The issue stemmed from fortified libc functions enforcing the original declared size of `d_name[NAME_MAX + 1]` despite the code’s attempt to reallocate the buffer. The fix converts `d_name` to a flexible array member and adjusts allocation calculations. Junio C Hamano suggested rewriting the test to avoid Perl, and Patrick Steinhardt proposed using `FLEX_ALLOC_MEM()` for cleaner memory management. The patch is **on track for merging** once the test is adjusted.

**Key detail:** macOS allows filenames to exceed `NAME_MAX` bytes because the system’s `dirent` structure uses `__DARWIN_MAXPATHLEN` (1024 bytes) as its limit, not `NAME_MAX`.

---

### `USE_NSEC` debate: build-time vs. runtime
The **`USE_NSEC` discussion** continued, with **D. Ben Knoble** confirming that nanosecond timestamps are preserved on XFS, reinforcing Jeff King’s earlier findings for ext4, ext2, and vfat. The thread remains stuck between three proposals:
1. **Knoble’s build-time parity patch** (queued by Junio).
2. **Brian M. Carlson’s default-flip to `true`** (with a config knob for edge cases).
3. **Patrick Steinhardt’s runtime-configuration idea** (e.g., `core.useNsec`), which Peff endorsed but noted lacks reliable auto-detection.

**Key detail:** Peff’s testing showed no reliable, portable way to detect whether a filesystem preserves nanosecond timestamps, complicating runtime configuration.

---

## In brief

**`git rev-list --exclude-first-parent-only` bugfix** -- Junio C Hamano fixed a misbehavior where the option incorrectly excluded commits when additional commits were explicitly specified. The patch adds an early return in `process_parents()` to prevent the exclusion logic from marking commits as uninteresting.

**`git rm -n *.json` pathspec clarification** -- Patrick Steinhardt explained that the recursion is expected behavior: `git rm` treats arguments as *pathspecs*, where `*` matches directory separators by default. The solution is to use `:(glob)` (e.g., `git rm -n ':(glob)*.json'`).

**`git replay --linearize` v6** -- Toon Claes’s series fixing a regression in `--linearize` (silent commit dropping) was **ready for final review** after addressing all critical post-merge issues. Junio C Hamano raised a minor documentation question about the `--linearize`/`--revert` incompatibility.

**`git refs` subcommand usability feedback** -- Toon Claes raised usability concerns about `--no-deref` in `git refs create` (for symref creation) and `git refs rename` (symref support). The series is already merged, but the feedback may inform future refinements.

**`git repo info` prefix query** -- K Jayatheerth’s GSoC patch added category-based prefix querying to `git repo info` (e.g., `git repo info layout` returns `layout.bare` and `layout.shallow`). Junio C Hamano suggested supporting glob patterns (e.g., `layout.*`) for greater flexibility.

**`git history` signed commits** -- Souma’s three-patch series taught `git history`’s `fixup`, `reword`, and `split` subcommands to sign commits via `commit.gpgsign` and `-S/--gpg-sign`. The series includes **regression tests** for config-driven signing and command-line overrides.

**`git diff --index` proposal** -- A patch proposing `--index` as a replacement for `--cached` received **surface-level support** from krobchai wongkamw, but no substantive review. The change aims to modernize Git’s CLI, though backward compatibility concerns remain unaddressed.

---

## On the radar

**`includeIf.worktree` symlink inconsistency** -- Patrick Steinhardt identified a behavioral discrepancy between the new `worktree` condition and the existing `gitdir` condition: `worktree` resolves symlinks to the real path, while `gitdir` matches both symlinked and real paths. The inconsistency could confuse users and may require a design decision before the series graduates to `next`.

**`git rebase --edit-commits` feature request** -- Matthias Beyer proposed a `--edit-commits` option for `git rebase -i` to programmatically mark commits for editing. The request was **resolved by demonstrating `GIT_SEQUENCE_EDITOR`**, but the discussion surfaced edge cases in shell-nesting and quoting that may inform future tooling improvements.

**ODB abstraction topics** -- Junio C Hamano acknowledged Patrick Steinhardt’s clarification that **ps/odb-drop-whence**, **ps/odb-generalize-prepare**, and **ps/refs-writing-subcommands** are ready for `next` after minor adjustments. These topics advance Git’s ODB abstraction effort, a key architectural goal.