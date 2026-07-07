## The day in brief.

June 29, 2026 was a **heavy, milestone-heavy day**—Git 2.55.0 shipped, the Git for Windows 2.55.0 installer followed hours later, and Junio’s “What’s cooking” report catalogued 505 commits from 100 contributors. **Two things you absolutely should not miss:** the **Git 2.55.0 release** (parallel hooks, Linux fsmonitor, ODB abstraction, Rust readiness) and **Patrick Steinhardt’s 12-patch reftable hardening series**, which landed fuzzing infrastructure and fixes for a dozen parser vulnerabilities.

---

## Notable threads

### Git 2.55.0 released
Junio C Hamano tagged the final tarball, summarizing 505 non-merge commits from 100 contributors. Highlights: parallel hooks (`hook.jobs`), new built-ins (`git format-rev`, `git url-parse`), Linux fsmonitor daemon, ODB abstraction, MIDX compaction, and Rust now enabled by default. The changelog is organized by UI, performance, and fixes; no regressions were reported in the thread.

---

### Reftable security hardening (ps/reftable-hardening)
Patrick Steinhardt’s **12-patch series** landed fuzzing infrastructure (libFuzzer + Meson) and fixes for out-of-bounds reads/writes, NULL derefs, and `abort()` calls in the reftable parser. All patches are now merged; Junio approved the majority as “obviously correct.” The fuzzer has run 2+ hours without surfacing new issues, and the fixes replace crashes with `REFTABLE_FORMAT_ERROR` returns. This is a **proactive security hardening** effort with no user-visible changes.

---

### ODB abstraction: `odb-drop-whence` (ps/odb-drop-whence)
Patrick Steinhardt’s **7-patch series** removes the `whence` field from `struct object_info`, replacing it with an opt-in `struct odb_source_info` that carries backend-specific data. The series is **merged into `next`**; Junio’s only nit was an implicit upcast in patch 3/7, which Patrick defended as technically correct. This is a foundational step toward pluggable ODB backends.

---

### `git history drop` (ps/history-drop)
Patrick Steinhardt’s **11-patch series** adds the `git history drop` subcommand, enabling commit removal with descendant replay. The series refactors the reset machinery to avoid touching HEAD/index unnecessarily, adds dry-run mode, and includes 561 lines of test coverage. Junio’s review surfaced a **critical ref-resolution flaw** in `find_head_tree_change()`; Patrick confirmed the fix for v8. The series is **ready for final review** and targets `next`.

---

### `git replay --linearize` (toon/replay-linearize)
Toon Claes’s **6-patch v6 series** adds `--linearize` to `git replay`, flattening merge commits into a linear history. The series is **architecturally resolved**—Junio approved the lookahead buffer redesign, and all failing test cases now pass. The interface debate (`--linearize` vs. `--rebase-merges=<mode>`) remains open but is no longer blocking. The series is **ready for final review** and targets `next`.

---

### Git for Windows 2.55.0
Johannes Schindelin announced the downstream installer, bundling upstream Git 2.55.0 with Windows-specific fixes: `git fetch` regression on older Windows 10, Git Bash process shutdown, parallel checkout with FSCache, and stricter `.git` file validation. The release drops Windows 8.1 support (following MSYS2) and includes SHA-256 checksums for all artifacts. This is a **downstream packaging effort** with no upstream patches.

---

## In brief

**Reftable compaction fix** — Patrick Steinhardt fixed a quadratic-time scalability issue in the reftable backend, reducing runtime from O(n²) to O(n) for bulk ref deletion/re-creation workflows.

**French translation update** — Jean-Noël Avila updated the French `.po` file to match the latest source strings.

**`git refs` subcommands** — Patrick Steinhardt’s **5-patch series** adding `create`, `delete`, `update`, and `rename` subcommands to `git refs` is **merged into `next`**. The series consolidates reference manipulation under a unified interface.

**`the_repository` removal** — Tian Yuchen’s **3-patch series** migrating `trust_executable_bit` into `repo_config_values` is **merged**. This is another incremental step in the ongoing `the_repository` removal effort.

**GSoC 2026: Partial clone pruning** — Siddharth Shrimali’s progress report (weeks 3–5) proposes `git maintenance run --task=gc-promisor` to safely prune unneeded promisor objects. The design is still open for feedback.

**`greplint.pl`** — Michael Montalbo’s **6-patch series** converting bare `grep` assertions to `test_grep` is **merged**. The series fixed 10+ pre-existing test bugs and added a linter to prevent regressions.

**`paint_down_to_common()` optimization** — Tian Yuchen and Kristofer Karlsson’s **10-patch series** optimizes merge-base computation by terminating early when one side’s commits are exhausted. The series is **merged into `next`** after addressing a procedural merge conflict.

**`USE_NSEC` debate** — D. Ben Knoble’s patch adding a `nanosec` Meson option sparked a **design discussion** about whether `USE_NSEC` should be runtime-configurable. Patrick Steinhardt proposed always compiling nanosecond support and exposing it via `core.useNsec`; no consensus yet.

**`git history squash`** — Harald Nordgren’s **5-patch v7 series** adds `git history squash` to fold a commit range into its oldest commit. The series is **ready for final review** after addressing all feedback, including template alignment with `git rebase -i`.

**CI alignment** — Patrick Steinhardt’s patch enabling `GIT_TEST_LONG` for GitLab CI remains **on hold** pending a policy decision about CI resource allocation. Junio suggested GitLab could run a superset of GitHub’s tests, but no concrete plan has emerged.

---

## On the radar

**`git history drop` ref-resolution flaw** — Junio’s review identified a **critical bug** in `find_head_tree_change()`; Patrick confirmed the fix for v8. The series is otherwise ready for final review.

**`git replay --linearize` interface debate** — The discussion about whether `--linearize` should mirror `git rebase`'s `--rebase-merges=<mode>` syntax remains open. No consensus yet, but the series is otherwise ready for final review.

**`USE_NSEC` runtime configuration** — Patrick Steinhardt’s proposal to make `USE_NSEC` runtime-configurable (via `core.useNsec`) is still under discussion. The original Meson parity patch remains queued for merging.

**GitLab CI visibility** — Junio raised a **policy question** about how to make GitLab CI results more visible to maintainers. No concrete plan yet, but Patrick’s badge addition to `README.md` was accepted as a low-cost improvement.