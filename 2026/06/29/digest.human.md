# The Git Mailing List Daily Digest for 2026/06/29

## The day in brief
A busy Monday with **106 emails across 29 threads**, dominated by **feature finalizations** and **critical regressions**. The `git history drop` and `git history squash` series reached completion, while a **post-merge regression in `git replay --linearize`** (silent commit dropping) emerged as the day’s most urgent issue. Architectural debates continued around **ODB abstraction** and **reftable hardening**, and Git 2.55.0 was officially released.

---

## Notable threads

### `git replay --linearize` regression: silent commit dropping
**Subject:** [PATCH v5 0/3] replay: introduce --linearize option
**Author:** Toon Claes

The `git replay --linearize` feature, merged in v5, **silently drops commits** when replaying a single branch containing merge commits. Johannes Schindelin identified the regression with a concrete test case (`master~2..master`), showing that v5 replays only the tip commit directly onto `--onto`, dropping intermediate commits. The root cause is the removal of `replayed_base` logic in `pick_regular_commit()`. Patrick Steinhardt and Toon Claes are now debating whether to revert v5 or redesign the interface to match `git rebase`'s `--rebase-merges` modes. **Action required before 2.55.1.**

---

### `git history drop` series reaches v7
**Subject:** [PATCH v7 0/11] history: add drop subcommand
**Author:** Patrick Steinhardt

The 11-patch series adding `git history drop` (remove a commit and replay descendants) is now **complete and ready for final review**. Key improvements in v7:
- Resolved code duplication by exposing `replay_result_queue_update()` as a shared helper.
- Modernized the reset API with explicit `UPDATE_HEAD` control and `oid_from` for flexible reset origins.
- Added 537 lines of test coverage for edge cases (bare repos, dry-run, conflicts).
- Fixed a latent bug in reset machinery (cache-tree updates now restricted to hard resets).

Junio identified a **logical flaw in `find_head_tree_change()`** (HEAD resolution mismatch when `--update-refs=head` is used), which Patrick will address in v8. The series is otherwise **technically sound and uncontroversial**.

---

### `git history squash` series finalized
**Subject:** [PATCH v6 0/4] history: add squash subcommand
**Author:** Harald Nordgren

The `git history squash` feature (fold a range into its oldest commit) is now **code-complete** after addressing all review feedback. Key design decisions:
- **Ref handling**: Rejects operations with refs pointing to interior commits by default, advising `--update-refs=head`.
- **Merge commits**: Rejects merges with external parents to avoid ambiguity.
- **`--reedit-message`**: Gathers all folded-in commit messages (oldest first) into an editor template, mirroring `git rebase -i`.

Junio requested **CLI documentation fixes** (option ordering in SYNOPSIS, `@` shorthand in examples), and Phillip Wood suggested **cleaner commit-message templates** (omitting `fixup!` noise). The series is **ready for merge** pending these cosmetic updates.

---

### Reftable security hardening (v2)
**Subject:** [PATCH v2 0/12] reftable: security hardening against corrupted files
**Author:** Patrick Steinhardt

A **12-patch series** hardening the reftable backend against maliciously corrupted files, discovered via libFuzzer. Key fixes:
- **Out-of-bounds reads/writes** in block parsing (patches 6–11).
- **NULL pointer dereferences** and **uninitialized memory usage** (patches 8–10).
- **Fuzzing infrastructure** (patches 1–2) with Meson support for libFuzzer.

The series is **technically complete**, with each fix paired with a unit test. Christian Couder’s feedback (test helper extraction) was incorporated in v2. **Ready for substantive review.**

---

### ODB abstraction: `struct object_info` refactoring
**Subject:** [PATCH v1 0/6] refactor `struct object_info` to use `source` field
**Author:** Patrick Steinhardt

The series replaces `struct object_info`'s `whence` field with a new `struct object_info_source` to enable multi-source object resolution. Junio and Justin Tobler raised **architectural questions** about where source-tracking logic should live (caller vs. backend initialization). The debate centers on whether the current opt-in design (passing `source` as a parameter) is preferable to a centralized approach (recording source during `packed_git` initialization). **No consensus yet; v2 expected.**

---

### `git merge-base` clock-skew regression fix
**Subject:** [PATCH 0/2] commit-reach: fix clock-skew regression in merge-base
**Author:** Kristofer Karlsson

A **two-patch bugfix** for a regression in `git merge-base` (without `--all`) introduced in 2025. The issue occurs in v1 commit-graph repositories with clock skew, where the correct merge base has a lower committer date than its ancestor. The fix gates an early-exit optimization on a `gen_ordered` flag. Junio approved merging the series as-is, with the more complex `kk/merge-base-exhaustion` topic to be rebased later. **Ready for `next`.**

---

## In brief
- **`greplint.pl` series**: Junio confirmed Gábor Szeder’s 2021 fix for `t3420` is already merged; v3 will audit `# lint-ok:` comments. **Test infrastructure cleanup.**
- **`git refs` subcommands**: Junio queued Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` subcommands. **Ready for `next`.**
- **`USE_NSEC` debate**: Brian M. Carlson and Patrick Steinhardt proposed flipping the default to `true` or converting it to runtime configuration. **No consensus; discussion ongoing.**
- **Git 2.55.0 released**: Junio announced the release, highlighting **parallel hooks**, **Rust support (enabled by default)**, and **Linux fsmonitor daemon**. **505 commits from 100 contributors.**
- **Git for Windows 2.55.0**: Johannes Schindelin announced the Windows release, dropping support for Windows 8.1 and fixing Windows-specific bugs. **Downstream packaging update.**

---

## On the radar
- **`git replay --linearize` regression**: Critical post-merge issue requiring a fix before 2.55.1.
- **`kk/merge-base-exhaustion`**: Kristofer Karlsson’s optimization series, now blocked on the clock-skew regression fix.
- **ODB abstraction follow-ups**: Justin Tobler’s `receive-pack` transaction series and Patrick Steinhardt’s `odb-drop-whence` series. **Architectural debates ongoing.**