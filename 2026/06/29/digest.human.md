The day in brief
2026/06/29 was a **heavy, milestone-heavy day**—Git 2.55.0 shipped, Git for Windows 2.55.0 followed, and the Git Merge 2026 CFP deadline was extended. The list saw **106 emails across 29 threads**, with **three threads dominating**: the Git 2.55.0 release announcement, the Git for Windows 2.55.0 release, and Junio’s “What’s cooking” report. **Two things you absolutely should not miss**: (1) the **Git 2.55.0 release** (505 commits, 100 contributors, new parallel hooks, Linux fsmonitor, ODB abstraction, Rust readiness), and (2) **Patrick Steinhardt’s reftable hardening series** (12 patches, fuzz-driven, now merged).

---

### Notable threads

#### Git v2.55.0 released
Junio C Hamano announced the final release of Git 2.55.0, integrating 505 non-merge commits from 100 contributors. The release introduces **parallel hooks**, new built-ins (`git format-rev`, `git url-parse`), a **Linux fsmonitor daemon**, performance optimizations (revision traversal, reachability bitmaps, sparse-index), and foundational work for **ODB abstraction** and **Rust support**. The changelog is organized into UI/workflow changes, performance improvements, internal refactoring, and over 50 bug fixes. **Key technical details**: new config keys (`hook.jobs`, `hook.<event>.enabled`, `remote.*.negotiationRestrict`), breaking changes (sideband terminal control sequences disabled by default, stricter proxy URL validation), and significant architectural shifts (ODB abstraction, Rust readiness). The release is a major milestone, reflecting active development across the project.

---

#### Git for Windows 2.55.0 released
Johannes Schindelin announced Git for Windows 2.55.0, a downstream packaging of upstream Git 2.55.0 with Windows-specific dependencies (MSYS2 runtime, Git Credential Manager, OpenSSL, Bash, cURL). This release drops Windows 8.1 support (following MSYS2) and includes Windows-specific fixes: a `git fetch` regression causing infinite unlink loops on older Windows 10, Git Bash process shutdown issues, parallel checkout failures with FSCache, stricter `.git` file validation (matching TortoiseGit 2.19), and improved handling of large clones/packfiles. SHA-256 checksums are provided for all artifacts (x86_64, ARM64, 32-bit installers/archives). This is a downstream release notice, editorially separate from core Git development.

---

#### Reftable hardening series (ps/reftable-hardening)
Patrick Steinhardt’s **12-patch series hardening Git’s reftable backend against maliciously corrupted files** is now **merged**. The series systematically addresses vulnerabilities discovered via fuzzing (libFuzzer), including out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. **Key technical details**: fuzzing infrastructure (Meson build support for libFuzzer, new oss-fuzz target), a test helper (`cl_reftable_write_block`), and individual fixes for each vulnerability, all paired with unit tests. The series is part of the ongoing reftable backend effort led by Patrick Steinhardt and Karthik Nayak. **Why it matters**: reftable files are not transferred over the network, but hardening them is a proactive defense-in-depth measure, especially as the backend matures and gains adoption.

---

#### ODB abstraction: `odb-drop-whence` (ps/odb-drop-whence)
Patrick Steinhardt’s **7-patch series refactoring `struct object_info` to replace the `whence` field with an opt-in `struct object_info_source`** is now **merged into `next`**. The series is part of the broader ODB abstraction effort, enabling multi-source object resolution and pluggable ODB backends. **Key technical details**: the `whence` field and `u` union are replaced with a new `source_infop` parameter, allowing backends to record which ODB source (loose, packed, or in-memory) an object was retrieved from. The series is foundational for future work, including backend-agnostic object writes. **Review context**: Junio C Hamano conceptually approved the series, and Justin Tobler provided substantive review, focusing on architectural design and future implications.

---

#### `git history drop` (ps/history-drop)
Patrick Steinhardt’s **11-patch series adding the `git history drop` subcommand** is now **complete and ready for final review**. The subcommand removes a commit from history and replays its descendants onto its parent, with a dry-run mode, refactoring of the reset machinery to avoid touching HEAD/index unnecessarily, and comprehensive test coverage. **Key technical details**: the series introduces new helpers (`index_state_unmerged_to_stage0`, `replay_result_queue_update`), modernizes the reset API (`reset_working_tree_flags` enum, `oid_from` field), and implements the `drop` subcommand with conflict detection, bare repository support, and preservation of local changes. **Review context**: Junio C Hamano identified a logical flaw in the `find_head_tree_change()` helper, which Patrick addressed in v7. The series is now technically complete and ready for integration.

---

#### `git history squash` (hn/history-squash)
Harald Nordgren’s **4-patch series adding the `git history squash` subcommand** is now **complete and ready for final review**. The subcommand folds a commit range into its oldest commit while preserving descendant history, with a `--reedit-message` flag (aligned with `git rebase -i`’s squash behavior), explicit rejection of ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`, and thorough test coverage. **Key technical details**: the series introduces new helpers (`resolve_squash_range`, `build_squash_message`), refactors the ref-update logic into a two-phase design (`compute_pending_ref_updates`/`apply_pending_ref_updates`), and documents the merge limitation (rejects merges with external parents). **Review context**: Phillip Wood raised usability concerns about the `--reedit-message` template and input validation, which Harald addressed in v6. The series is now technically complete and ready for integration.

---

#### `git refs` subcommands (ps/refs-writing-subcommands)
Patrick Steinhardt’s **5-patch series adding `git refs create|delete|update|rename` subcommands** is now **merged into `next`**. The series consolidates reference manipulation functionality (previously scattered across `git-update-ref`, `git-symbolic-ref`, and other commands) under a unified `git refs` interface. **Key technical details**: the subcommands mirror the behavior of their legacy counterparts, with atomicity, backwards compatibility, and ergonomic improvements. **Review context**: Junio C Hamano approved the series, and Toon Claes provided surface-level review, focusing on usability and consistency. **Why it matters**: this is a step toward reducing fragmentation in Git’s reference-handling commands and improving discoverability.

---

### In brief

- **`git replay --linearize` (tc/replay-linearize)**: Toon Claes’s 3-patch series adding `--linearize` to `git replay` is **under review**, with an **interface design debate** about whether to mirror `git rebase`'s `--rebase-merges=<mode>` syntax. The series is technically complete and addresses all prior feedback, including a regression fix for `--linearize` behavior when replaying a single branch containing merge commits.
- **Memoized commit traversal optimization (tamir/contains-optim)**: Tamir Duberstein’s 3-patch series optimizing `paint_down_to_common()` for one-sided histories is **ready for merging**, with all prior feedback addressed. The series extends Git’s memoized commit-traversal optimization from `git tag --contains` to all ref-filter commands, with robust handling of cycles from replacement refs.
- **`greplint.pl` (mm/greplint)**: Michael Montalbo’s 6-patch series introducing `greplint.pl` (a test-suite linter converting bare `grep` assertions to `test_grep`) is **merged**, with all prior feedback addressed. The series addresses systemic risks in automated conversion masking pre-existing test bugs and is now technically complete.
- **`git history drop` follow-up**: Patrick Steinhardt’s v7 of the `git history drop` series is **complete**, addressing all prior feedback, including a logical flaw in the `find_head_tree_change()` helper identified by Junio C Hamano. The series is now ready for final review.
- **`git refs` subcommands follow-up**: Junio C Hamano approved Patrick Steinhardt’s 5-patch series adding `git refs create|delete|update|rename` subcommands, with a minor typo fix in the `git refs create` error message. The series is now **merged into `next`**.
- **`paint_down_to_common()` optimization (kk/merge-base-exhaustion)**: Kristofer Karlsson’s 10-patch series optimizing `paint_down_to_common()` for one-sided histories is **ready for merging**, with all prior feedback addressed. The series introduces a lookahead buffer to resolve fragility in the v6 peek-based abstraction, addressing failing test cases and eliminating the need for defensive checks.
- **Shell completion dotfile hiding (zakariyah/shell-completion-dotfiles)**: Zakariyah Ali’s 3-patch series hiding dotfiles in shell completion is **blocked on structural concerns**, with Junio C Hamano requesting a two-patch progression (refactor AWK script for readability, then implement dotfile hiding).
- **`trust_executable_bit` migration (tian/trust-executable-bit)**: Tian Yuchen’s 3-patch series migrating `trust_executable_bit` into `repo_config_values` is **merged**, completing another step in the `the_repository` removal effort.
- **Priority queue optimization (kristofer/prio-queue-cascade)**: Kristofer Karlsson’s 2-patch series optimizing the sift-down algorithm in `prio_queue_get()` is **ready for merging**, with maintainer-approved consensus on a hybrid approach combining both optimizations contextually.
- **`git history drop` (merged)**: Patrick Steinhardt’s 11-patch series adding the `git history drop` subcommand is **merged**, with all prior feedback addressed, including a critical ref resolution fix.
- **`greplint.pl` (merged)**: Michael Montalbo’s 6-patch series introducing `greplint.pl` is **merged**, with all prior feedback addressed, including a fix for a latent lexer bug.
- **`git replay --linearize` (under review)**: Toon Claes’s 3-patch series adding `--linearize` to `git replay` is **under review**, with an **interface design debate** about whether to mirror `git rebase`'s `--rebase-merges=<mode>` syntax.
- **Memoized commit traversal optimization (merged)**: Tamir Duberstein’s 3-patch series optimizing `paint_down_to_common()` for one-sided histories is **merged**, with all prior feedback addressed.
- **macOS CI hang fix (merged)**: Jeff King’s 2-patch series fixing macOS CI hangs in `t5551` and `t5559` is **merged**, addressing the root cause (Apache’s `mod_http2` bug 70131) by increasing Apache’s `Timeout` directive and isolating the expensive test case.
- **`USE_NSEC` build option (dk/meson-enable-use-nsec-build)**: D. Ben Knoble’s patch adding a `nanosec` Meson build option to mirror Autotools’ `USE_NSEC` is **queued for merging**, with a **design debate** about whether to flip the default, deprecate the knob, or convert it to runtime-configurable.
- **ODB transaction refactoring (jt/receive-pack-use-odb-transactions)**: Justin Tobler’s 6-patch series refactoring `git-receive-pack` to use ODB transactions is **ready for v2**, with all prior feedback addressed, including error-handling improvements and lazy directory creation investigation deferred.
- **`excludes_file` migration (tian/excludes-file)**: Tian Yuchen’s 2-patch series migrating `excludes_file` into `repo_config_values` is **merged**, completing the three-phase guardrail migration and fulfilling Junio C Hamano’s requirement for merging.
- **`git history --reword` file stream leak fix (merged)**: Junio C Hamano’s patch fixing a file stream leak and inefficient file handling in `git history --reword` is **merged**, with all prior feedback addressed, including a reliability concern raised by Patrick Steinhardt.
- **Security inquiry: CVE-2026-34182 (closed)**: A reported critical CVE in OpenSSL 3.5.6 prompted an inquiry about Git’s exposure, which concluded that neither Git nor Git for Windows is exposed due to limited OpenSSL usage. The thread is **closed**.
- **Reftable leak fix (jk/reftable-leakfix)**: Jeff King’s patch fixing a memory leak in the reftable backend is **merged**, with a trivial conflict resolution noted.
- **`git receive-pack` hang on zero-object push (closed)**: Wei Hu reported a hang in `git receive-pack` on zero-object pushes to promisor remotes, which was **already fixed** by commit d9982e8290. The thread is **closed**.
- **Clock-skew regression in merge-base (merged)**: Kristofer Karlsson’s 2-patch series fixing a clock-skew regression in `git merge-base` is **merged**, with all prior feedback addressed, including a proposal to merge the series as-is for the 2.55 release.
- **Off-topic link-exchange solicitation (spam)**: An off-topic solicitation for a link exchange is **ignored**.
- **Git Merge 2026 CFP deadline extension**: Taylor Blau announced an extension of the Git Merge 2026 Call for Proposals deadline from June 30, 2026 to July 14, 2026 at 11:59 PM (UTC-7).

---

### On the radar
- **`git replay --linearize` interface design**: The debate about whether to mirror `git rebase`'s `--rebase-merges=<mode>` syntax is unresolved. Toon Claes and Johannes Schindelin argue for `--linearize` as a standalone flag, while Patrick Steinhardt proposes adopting `git rebase`'s syntax for consistency.
- **`USE_NSEC` build option**: The design debate about whether to flip the default, deprecate the knob, or convert it to runtime-configurable is unresolved. D. Ben Knoble’s patch is queued for merging, but the broader discussion continues.
- **ODB transaction refactoring**: Justin Tobler’s 6-patch series refactoring `git-receive-pack` to use ODB transactions is ready for v2, with error-handling improvements and lazy directory creation investigation deferred.
- **`git history squash` default behavior**: Phillip Wood’s suggestion that `--reedit-message` (or `--edit`) should be the default to encourage commit hygiene is unresolved. The series is otherwise complete and ready for final review.