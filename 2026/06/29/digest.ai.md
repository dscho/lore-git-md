The day in brief
**2026/06/29** was a **heavy, milestone-heavy day**—Git 2.55.0 shipped, Git for Windows 2.55.0 followed, and the Git Merge 2026 CFP deadline was extended. The list saw **106 emails across 29 threads**, with **two major releases** and **a flurry of last-minute bugfixes, security hardening, and ODB refactoring** dominating the traffic. The one thing a reader absolutely should not miss: **Git 2.55.0 is out**, carrying **parallel hooks, new built-ins, Linux fsmonitor, and the first wave of Rust readiness**—and the reftable backend is now **fuzz-hardened** against corrupted files.

---

### Notable threads

#### Git v2.55.0 released
Junio C Hamano’s announcement of Git 2.55.0 (505 non-merge commits from 100 contributors) was the day’s headline. The release integrates **parallel hooks** (`hook.jobs`), two new built-ins (`git format-rev` and `git url-parse`), the **Linux fsmonitor daemon** (bringing Linux to parity with Windows/macOS), and **Rust readiness** (now enabled by default, though still opt-out). Performance wins include **revision traversal optimizations** (100–1000× speedups for asymmetric queries), **reachability bitmaps**, and **sparse-index improvements**. The changelog also notes **over 50 bugfixes**, including a **Windows-specific `git fetch` regression** that caused infinite unlink loops on older Windows 10. The thread drew lighthearted commentary about a humorous contributor list entry (“Claude Sonnet 4.6”) but no substantive discussion—this was a pure release notice.

---

#### Git for Windows 2.55.0
Johannes Schindelin followed with the Git for Windows 2.55.0 announcement, a downstream packaging of upstream Git 2.55.0 with Windows-specific dependencies (MSYS2, Git Credential Manager, OpenSSL, Bash, cURL). This release **drops Windows 8.1 support** (following MSYS2) and includes **Windows-specific fixes**: a `git fetch` regression causing infinite unlink loops on older Windows 10, Git Bash process shutdown issues, parallel checkout failures with FSCache, stricter `.git` file validation (matching TortoiseGit 2.19), and improved handling of large clones/packfiles. SHA-256 checksums are provided for all artifacts (x86_64, ARM64, 32-bit installers/archives). The thread is editorially separate from upstream Git but critical for Windows users.

---

#### Reftable security hardening (ps/reftable-hardening)
Patrick Steinhardt’s **12-patch series** systematically hardens Git’s reftable backend against maliciously corrupted files, addressing **out-of-bounds reads/writes, NULL pointer dereferences, uninitialized memory usage, and calls to `abort()`**. The series includes **fuzzing infrastructure** (libFuzzer + Meson support) and a **test helper** (`cl_reftable_write_block`) to reduce boilerplate in unit tests. Junio C Hamano approved patches 6–11/12 as “obviously correct,” and Christian Couder approved patch 5/12. The series is **technically complete and ready for merging**, with no unresolved feedback. The fuzzer has run for 2+ hours without surfacing new issues, and the fixes replace crashes with graceful error returns (`REFTABLE_FORMAT_ERROR`). This is a **proactive security effort** with no user-visible changes, targeting a backend that is not yet the default but is under active development.

---

#### ODB abstraction: `odb-drop-whence` (ps/odb-drop-whence)
Patrick Steinhardt’s **7-patch series** refactors `struct object_info` to replace the coarse `whence` field and `u` union with an opt-in `struct odb_source_info`, enabling multi-source object resolution for pluggable ODB backends. The series is **conceptually approved and queued for merging** by Junio, with all prior feedback addressed. Justin Tobler’s review raised **architectural questions** about whether the ODB source should be tracked at `struct packed_git` initialization time rather than passed as an optional parameter, but the discussion remains open. The series is **self-contained and targets `master`**, with no behavior changes—it’s a foundational step for the ODB abstraction effort.

---

#### `git history drop` (ps/history-drop)
Patrick Steinhardt’s **11-patch series** adds the `git history drop` subcommand, allowing users to remove a commit and replay its descendants onto its parent. The series includes **dry-run mode**, **refactored reset machinery** (to avoid touching HEAD/index unnecessarily), and **561 lines of test coverage**. Junio C Hamano’s review identified a **logical flaw** in the `find_head_tree_change()` helper, which could incorrectly conclude that HEAD does not move when `--update-refs=head` is used. The fix is straightforward and will appear in v8. The series is **ready for final review**, with all other feedback addressed.

---

#### `git history squash` (hn/history-squash)
Harald Nordgren’s **4-patch series** introduces `git history squash`, a subcommand to fold a commit range into its oldest commit while replaying descendants on top. The series is **code-complete and ready for Junio’s final review**, with all prior feedback addressed, including **input validation**, **merge commit handling**, and **`--reedit-message` template alignment** with `git rebase -i`. Phillip Wood’s review raised **usability questions** about whether `--reedit-message` should be the default (for commit hygiene) and **recoverability concerns** when `--update-refs` moves multiple branches. Junio’s feedback focused on **CLI/documentation style** (option ordering, `@` shorthand), which Harald addressed in v7. The series is **uncontroversial and well-tested**, with 747 lines of test coverage.

---

#### `git replay --linearize` (toon/replay-linearize)
Toon Claes’s **3-patch series** (v6) adds a `--linearize` option to `git replay` to flatten merge commits, producing a linear history. The series is **technically complete**, with all prior feedback addressed, including a **regression fix** for commit-dropping when replaying a single branch containing merge commits. The **interface design debate** (whether to mirror `git rebase --rebase-merges=<mode>`) remains unresolved but is no longer blocking. Junio C Hamano’s review suggested **justifying the `--linearize`/`--revert` incompatibility** or moving the note to a BUGS/LIMITATIONS section. The series is **ready for final review**, with no unresolved technical objections.

---

### In brief

- **Reftable leak fix (jk/reftable-leakfix)** – Jeff King fixed a memory leak in `reftable_writer_new` where the `reftable_writer` struct was allocated before input validation. The patch is **merged**, with a trivial conflict resolution noted.
- **ODB transactions in `receive-pack` (jt/receive-pack-use-odb-transactions)** – Justin Tobler’s **6-patch series** refactors `git-receive-pack` to use ODB transactions instead of `tmp_objdir`. The series is **ready for v2**, with all feedback addressed, including **error-handling improvements** and **lazy directory creation investigation deferred**.
- **`excludes_file` libification (tian/excludes-file)** – Tian Yuchen’s **3-patch series** migrates the global `excludes_file` variable into `struct repo_config_values` as part of the libification effort. The series is **merged into `next`**, with the **three-phase guardrail migration** fully executed.
- **`git history --reword` file stream leak (junio/history-reword-fix)** – Junio C Hamano’s **bugfix patch** addresses a file stream leak and inefficient file handling in `fill_commit_message()`. The patch is **ready for merging**, with Patrick Steinhardt’s feedback about **error-checking** addressed in v2.
- **OpenSSL CVE-2026-34182 inquiry** – A reported critical CVE in OpenSSL 3.5.6 (bundled with Git 2.54.0 and Git for Windows) prompted an inquiry, but the thread **closed** after confirming Git and Git for Windows are not exposed. The next Git for Windows release (v2.55.0) will include OpenSSL 3.5.7.
- **`git receive-pack` hang on zero-object push to promisor remote** – Wei Hu reported a hang in `git receive-pack` when pushing a zero-object ref update to a promisor-configured repository. Patrick Steinhardt confirmed the bug was **already fixed** by commit d9982e8290, which will ship in Git 2.55.0.
- **Clock-skew regression in `git merge-base` (kk/merge-base-clock-skew-fix)** – Kristoffer Karlsson’s **2-patch series** fixes a regression in `git merge-base` (without `--all`) introduced by 93e5b1680e. The series is **ready for merging**, with Junio’s integration question resolved.
- **Git Merge 2026 CFP deadline extended** – Taylor Blau extended the Git Merge 2026 Call for Proposals deadline from June 30 to July 14, 2026, to accommodate more contributors.

---

### On the radar
- **`kk/merge-base-exhaustion` (v5)** – Kristoffer Karlsson’s **10-patch series** optimizes merge-base computation by stopping early when one side’s exclusive commits are exhausted. The series is **ready for merging**, with all feedback addressed, but will be rebased on top of the **clock-skew regression fix** (`kk/merge-base-clock-skew-fix`).
- **`tb/repack-geometric-cruft` (11 patches)** – Taylor Blau’s series teaches `git repack --geometric --cruft` to roll up non-cruft packs into a geometric progression while writing a separate cruft pack for unreachable objects. The series is **new and needs review**.
- **`ps/libgit-in-subdir` (4 patches)** – Patrick Steinhardt’s series moves all `libgit.a` source files into a new `lib/` directory to clean up the top-level directory. The series is **new and needs review**.
- **`ps/cat-file-remote-object-info` (4 patches)** – Patrick Steinhardt’s series adds `remote-object-info` to `git cat-file --batch-command`, allowing clients to request object metadata from a remote server via protocol v2. The series is **new and needs review**.
- **`mm/diff-process-hunks` (1 patch)** – Max Mustermann’s series introduces `diff.<driver>.process` config to let external long-running processes act as hunk providers. The series is **new and needs review**.