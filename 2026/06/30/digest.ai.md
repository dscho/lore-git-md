Here is the digest for June 30, 2026:

---

## The day in brief

June 30 was a **busy, milestone-heavy day** on the Git mailing list, with **94 emails across 23 threads**. The standout developments: **Patrick Steinhardt’s ODB abstraction work reached key integration milestones**, with two series (`ps/odb-generalize-prepare` and `ps/odb-drop-whence`) now approved and queued for `next`. **Phillip Wood’s systemic rebase-dropped-commit fix** (11 patches) is complete and ready for final review, addressing a long-standing bug where notes from dropped commits were incorrectly copied to HEAD. **Junio C Hamano merged several uncontroversial series**, including Patrick’s reftable hardening and reference-writing subcommands, while **usability discussions** (e.g., `git blame -b` output formatting, `git history squash` recoverability) highlighted recurring pain points in Git’s UX. The day also saw **CI and build system improvements**, with Jeff King’s memory leak fixes and a proposal to streamline leak detection in Git’s CI pipeline.

---

## Notable threads

### ODB abstraction reaches integration milestones
Two series from **Patrick Steinhardt**—`ps/odb-generalize-prepare` (3 patches) and `ps/odb-drop-whence` (7 patches)—were **approved and queued for `next`**, marking significant progress in the ODB abstraction effort. The former generalizes the `reprepare()` callback into a `prepare()` callback with an optional flush flag, enabling `git grep` to work with pluggable ODB backends. The latter refactors `struct object_info` to use a `source` field, replacing the coarse `whence` field and paving the way for multi-source object resolution. Both series are now **technically complete**, with all review feedback addressed, and will graduate to `master` after cooking in `next`. The approvals reflect the project’s growing confidence in the ODB abstraction layer, which aims to make Git’s object storage backend-agnostic.

### Systemic fix for rebase-dropped commits
**Phillip Wood** posted an **11-patch series** that overhauls how the sequencer handles dropped commits during rebase, fixing a bug where notes from dropped commits were incorrectly copied to the current HEAD. The series introduces a `PICK_RESULT_DROPPED` enum member and modifies `pick_one_commit()` to skip `record_in_rewritten()` for dropped commits, ensuring notes and post-rewrite hooks no longer receive misleading information. The fix addresses **four additional rebase bugs** (external merge strategy failures, command execution failures, final fixup cleanup, and edit command handling) and is **ready for final review**. Junio C Hamano has queued the series with minor typofixes, and the patches carry **Uwe Kleine-König’s Tested-by**, confirming the core regression is resolved. This is a **substantive, tested-level contribution** that refactors the sequencer’s control flow while addressing systemic technical debt.

### `git history squash` usability debate
A **user experience discussion** about `git history squash` (part of Harald Nordgren’s series) expanded into a broader critique of Git’s recoverability and reflog design. **Matt Hunter** and **Phillip Wood** highlighted that `git reset --hard` is insufficient to undo operations affecting multiple refs (e.g., when `--update-refs` moves other branches), as Git’s reflog lacks visibility into which refs were modified. **Harald Nordgren** proposed a "human-centered reflog" with visual hierarchy to reflect workflow structure, while **Phillip Wood** suggested **reflog transaction IDs** or a separate operations log to enable atomic undo. The discussion underscores a **systemic gap in Git’s design**: the reflog was never intended to support undo, and addressing this may require architectural changes. The thread remains open-ended, with no concrete patches yet, but signals growing attention to Git’s UX beyond individual commands.

### Reftable hardening series merged
**Patrick Steinhardt’s 12-patch series** hardening Git’s reftable backend against corrupted files was **merged into `next`**. The series fixes **out-of-bounds reads/writes, NULL pointer dereferences, and calls to `abort()`** during reftable parsing, discovered via libFuzzer. It also adds **fuzzing infrastructure** to Git’s build system, marking the first time the project has integrated coverage-guided fuzzing. The fixes are **backward-compatible** and replace crashes with error returns, improving robustness without altering on-disk formats. Junio C Hamano approved the majority of the patches as "obviously correct," and the series is now **ready for the next release cycle**. This is a **critical security hardening effort** for the reftable backend, which is increasingly used in large repositories.

### Memory leak fixes and CI improvements
**Jeff King** posted a **two-patch series** fixing memory leaks in `git format-patch` and the test harness. The `format-patch` leak (a `rev_info` struct not freed in `prepare_bases()`) had existed since 2016 but only became detectable after other leaks were fixed. The test harness fix redirects LeakSanitizer (LSan) output to stderr, preventing TAP parse errors when running tests under `prove`. The series is **low-risk and self-contained**, with both patches merged in principle. **Patrick Steinhardt** proposed enabling LSan for the `linux-TEST-vars` CI job to improve leak detection, benchmarking the overhead at ~3x slowdown. The discussion highlights the project’s **ongoing effort to improve memory hygiene**, with follow-up work planned for OpenSSL SHA-1/SHA-256 leaks and bloom filter leaks.

---

## In brief

**`git refs` subcommands merged** -- Patrick Steinhardt’s five-patch series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` was **merged into `next`**. The series consolidates reference manipulation under a unified interface, improving discoverability without altering on-disk formats. Junio C Hamano approved the final v3 reroll, which fixed a typo in the `git refs create` error message.

**`git blame -b` output formatting fix** -- René Scharfe posted a **patch** fixing a usability friction in `git blame -b`, where non-boundary commit hashes were one nibble longer than the user’s `core.abbrev` setting due to an unused caret marker. The patch refactors the mark-handling logic to count and print marks only when shown, aligning the output with `core.abbrev`. **Laszlo Ersek** confirmed the fix resolves the issue, and the patch is **ready for integration pending test coverage**.

**`git history --reword` file stream leak fixed** -- Junio C Hamano posted a **v2 patch** fixing a file stream leak in `git history --reword` that could cause issues on Windows. The patch ensures the file stream is closed before launching the external editor and restores explicit error checking on the initial `fwrite()` call. **Patrick Steinhardt** approved the changes, noting the commit message (critical data) will either be fully written or trigger a `die()`.

**Test modernization in `t7412-submodule-absorbgitdirs.sh`** -- Bryan B. Lima’s patch replacing raw test primitives (`test -f`, `test -d`) with descriptive helpers (`test_path_is_file`, `test_path_is_dir`) was **queued by Junio C Hamano**. The change improves test failure messages without altering behavior and is part of the ongoing community-wide test modernization effort.

**`USE_NSEC` debate continues** -- The discussion about whether to convert `USE_NSEC` from a build-time option to a runtime-configurable setting (e.g., `core.useNsec`) remains unresolved. **Jeff King** highlighted the lack of reliable auto-detection for filesystem timestamp granularity, while **Patrick Steinhardt** proposed always compiling nanosecond support into Git and exposing it via a runtime config knob. The debate centers on **trade-offs between flexibility and simplicity**, with no patch yet submitted for runtime configuration.

**`git clone` sparse-checkout RFC** -- Pushkar Singh’s RFC proposing `--only`/`--except` options for `git clone` to specify sparse-checkout paths received **feedback from Jeff King**, who suggested a file-based patterns approach instead of new CLI options. The discussion remains at the proposal stage, with no clear consensus on the interface or whether the feature addresses a real pain point.

---

## On the radar

**`git replay --linearize` interface design** -- The debate over whether `git replay --linearize` should mirror `git rebase`’s `--rebase-merges=<mode>` syntax remains unresolved. **Johannes Schindelin** opposes the idea on usability grounds, while **Patrick Steinhardt** requires explicit justification for any divergence. The series (v6) is **technically complete** but blocked on this design question.

**Reftable quadratic-time behavior** -- Jeff King identified a **quadratic-time scalability issue** in the reftable backend during bulk ref deletion and re-creation, where runtime grows quadratically with the number of deleted refs. The issue stems from tombstone iteration inefficiency, but **no patch has been proposed yet**. This could inform future reftable optimizations, particularly for large repositories.

**CI resource allocation** -- The proposal to enable `GIT_TEST_LONG` for GitLab CI remains **on hold pending further discussion** about whether GitLab should duplicate GitHub’s coverage or focus on additional tests/events. Junio C Hamano suggested GitLab could run a **superset** of GitHub’s tests, but no concrete plan has been proposed beyond the recently merged GitLab CI badge addition.