Here’s the daily digest for **June 30, 2026**, covering the Git mailing list traffic:

---

### The day in brief
A busy but focused day on the Git mailing list, with **94 emails across 23 threads**. The standout developments: **Phillip Wood’s 11-patch series fixing rebase-dropped-commit bugs** (including notes and post-rewrite hooks) is now complete and queued for merging, while **Patrick Steinhardt’s ODB abstraction work** continues to advance with multiple series landing or nearing readiness. A **memory leak fix in `git format-patch`** and **reftable hardening patches** also moved forward, alongside usability discussions about `git blame -b` and sparse-checkout in `git clone`. The day’s tone was **collaborative and milestone-heavy**, with several long-running efforts reaching resolution.

---

### Notable threads

#### **Rebase-dropped-commit bugs fixed**
**Thread**: *[PATCH 0/11] sequencer: avoid copying notes from dropped commits during rebase*
**Author**: Phillip Wood
**Status**: **Complete and queued** (Junio Hamano)
**Summary**: Phillip Wood’s 11-patch series overhauls how the sequencer handles dropped commits during rebase, fixing a long-standing bug where notes from dropped commits were incorrectly copied to the current HEAD. The series also addresses systemic issues like external merge strategy failures and command execution failures, and introduces a new `PICK_RESULT_DROPPED` enum to clarify control flow. Junio queued the series after resolving a minor `b4` tooling issue, calling it "obviously correct." The patches touch `sequencer.c` and add test coverage in `t3400-rebase.sh` and `t5407-post-rewrite-hook.sh`.
**Why it matters**: This is a **substantive, tested-level fix** that eliminates a source of repository corruption (misattributed notes) and improves the robustness of rebase operations. The series also lays groundwork for future sequencer improvements by simplifying control flow.

---

#### **ODB abstraction advances**
**Thread**: *[PATCH v2 0/2] odb: generalize reprepare() callback for pluggable ODBs*
**Author**: Patrick Steinhardt
**Status**: **Merged to `next`** (Junio Hamano)
**Summary**: Patrick Steinhardt’s series generalizes the `reprepare()` callback into a more flexible `prepare()` callback with an optional flush flag, enabling `git grep` to work with pluggable ODB backends. The patches replace `odb_source_packed_prepare()` with a backend-agnostic API and introduce `enum odb_prepare_flags` for cache control. Toon Claes’s substantive review concerns were resolved, and Junio explicitly approved the series. This is part of Patrick’s broader ODB abstraction effort, which aims to make Git’s object storage layer pluggable.
**Why it matters**: This is a **foundational refactoring** that removes a direct dependency on the "files" backend, advancing the goal of pluggable ODBs. The changes are mechanical but critical for future backend-agnostic features.

---

#### **`git history squash` usability debate**
**Thread**: *[PATCH 0/5] replay: introduce --linearize option*
**Author**: Harald Nordgren
**Status**: **v7 posted, ready for final review** (Junio Hamano)
**Summary**: Harald Nordgren’s `git history squash` feature (which folds a range of commits into one) is now complete after addressing all prior feedback, including interface design, merge commit handling, and `--reedit-message` template formatting. The series uses `git rebase -i`’s squash-message template for consistency and rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!`. Phillip Wood and Matt Hunter raised **usability questions** about default behavior (should `--reedit-message` be the default?) and recoverability (how to undo operations affecting multiple refs). Harald deferred these to follow-up work, noting the series is otherwise ready for merging.
**Why it matters**: This is a **user-facing feature** that simplifies monorepo workflows by avoiding the repeated conflict stops of a rebase-based approach. The debate about defaults and recoverability highlights broader UX challenges in Git’s interactive commands.

---

#### **Memory leak fixes land**
**Thread**: *[PATCH 0/2] fix memory leaks in format-patch and test harness*
**Author**: Jeff King (Peff)
**Status**: **Merged in principle** (Junio Hamano)
**Summary**: Peff fixed two small but long-standing memory leaks: one in `git format-patch` (a `rev_info` struct leaked since 2016) and another in the test harness (LSan error output misdirected to stdout). The patches are mechanical and low-risk, touching `builtin/log.c` and `t/test-lib.sh`. Patrick Steinhardt proposed enabling LSan for the `linux-TEST-vars` CI job to catch similar leaks earlier, and Junio endorsed the minimalist CI approach.
**Why it matters**: These are **targeted cleanups** that improve Git’s memory hygiene without disrupting existing workflows. The discussion also underscores the project’s growing investment in leak detection infrastructure.

---

#### **Reftable hardening patches**
**Thread**: *[PATCH v3 0/12] reftable: security hardening against corrupted files*
**Author**: Patrick Steinhardt
**Status**: **6/12 patches approved** (Junio Hamano)
**Summary**: Patrick Steinhardt’s 12-patch series hardens Git’s reftable backend against maliciously corrupted files, fixing out-of-bounds reads/writes, NULL pointer dereferences, and `abort()` calls. The series also adds a libFuzzer-based fuzzer to prevent regressions. Junio approved 6 of the 12 patches as "obviously correct," and the fuzzing infrastructure is now integrated into Git’s CI. The patches touch `reftable/` and add unit tests in `t/unit-tests/`.
**Why it matters**: This is a **proactive security effort** that addresses potential attack vectors in the reftable backend, which is increasingly used in large repositories. The fuzzing infrastructure is a first for Git and may be reused for other subsystems.

---

#### **`git blame -b` usability fix**
**Thread**: *git blame -b output formatting and commit hash abbreviation*
**Author**: René Scharfe (patch), Laszlo Ersek (report)
**Status**: **Patch posted, confirmed by reporter**
**Summary**: René Scharfe posted a patch to fix a usability friction in `git blame -b`: the command reserved an extra hex digit for a caret marker that was never displayed, causing commit hashes to exceed the user’s `core.abbrev` setting. The patch refactors the mark-handling logic in `blame.c` to count and print marks only when they’re shown, aligning the output with `core.abbrev`. Laszlo Ersek confirmed the fix resolves his workflow issue (manual truncation when pasting hashes into `git rebase -i`).
**Why it matters**: This is a **small but impactful UX improvement** that eliminates a persistent annoyance in manual workflows. The patch is well-scoped and unlikely to be controversial.

---

### In brief
- **`git refs` subcommands merged**: Patrick Steinhardt’s series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is now merged to `next`. The series consolidates reference manipulation under a unified interface and aligns with the ongoing `the_repository` removal effort.
- **Reftable quadratic-time fix reconsidered**: Kristofer Karlsson’s patch fixing quadratic-time behavior in the reftable backend (exposing tombstones to iterator bounds checks) is now under reconsideration due to limited real-world impact. Patrick Steinhardt identified a reproducible scenario (bulk ref deletion without compaction) that could demonstrate the patch’s value.
- **`git clone` sparse-checkout RFC**: Pushkar Singh proposed adding `--only`/`--except` options to `git clone` for sparse-checkout paths, but Jeff King suggested a file-based patterns approach instead. The discussion remains at the design stage, with no clear consensus yet.
- **`USE_NSEC` runtime configuration proposed**: Patrick Steinhardt proposed converting the `USE_NSEC` build-time option to a runtime config knob (e.g., `core.useNsec`), but Jeff King noted the lack of reliable auto-detection. The discussion is ongoing, with no patch yet.
- **Test modernization**: Bryan B. Lima’s patch replacing raw test calls in `t7412-submodule-absorbgitdirs.sh` with descriptive helpers was queued by Junio Hamano. This is part of the ongoing community-wide effort to modernize Git’s test suite.

---

### On the radar
- **`git replay --linearize` interface debate**: Toon Claes and Johannes Schindelin continue to debate whether `--linearize` should mirror `git rebase`’s `--rebase-merges=<mode>` syntax or remain a standalone flag. The discussion is now focused on documenting the trade-offs.
- **CI resource allocation**: The proposal to enable `GIT_TEST_LONG` for GitLab CI remains on hold pending further discussion about whether GitLab should duplicate GitHub’s coverage or focus on additional tests.
- **Rustification linkage**: Brian m. carlson opposed Jan Palus’s patch to remove the `$(LIB_FILE)` dependency from `$(RUST_LIB)`, explaining that the Rust code links against C functions from `libgit.a` during `cargo test`. The Meson build system’s lack of this dependency is now considered an oversight.

---

### Editorial note
Today’s traffic was **heavy on foundational work**—ODB abstraction, reftable hardening, and rebase robustness—while also addressing **user-facing pain points** like `git blame -b` and `git history squash`. The collaborative tone was notable, with reviewers and authors engaging deeply on design trade-offs (e.g., `USE_NSEC` runtime configuration) and usability concerns (e.g., `git history squash` defaults). The **rebase-dropped-commit series** stands out as a milestone, resolving a long-standing bug while improving the sequencer’s architecture. Meanwhile, the **ODB abstraction effort** continues to make steady progress, with Patrick Steinhardt’s patches landing or nearing readiness. The day’s discussions reflect Git’s dual focus on **internal refactoring** and **user experience**, with a growing emphasis on memory safety and security.