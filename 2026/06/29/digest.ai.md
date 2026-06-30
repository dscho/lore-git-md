# The Git Project Mailing List Daily Digest

**The day in brief.**
June 29, 2026 was a milestone-heavy day for Git development. The **Git 2.55.0 release** shipped, capping a cycle that introduced parallel hooks, new built-ins (`git format-rev`, `git url-parse`), Linux fsmonitor, and foundational ODB abstraction work. The mailing list was dominated by **post-release integration updates**—Junio’s "What’s cooking" report listed nine new topics needing review, while several long-running series (reftable hardening, ODB refactoring, `git history` subcommands) reached technical consensus. A **critical regression** in `git replay --linearize` was identified post-merge, and a **security-hardening series** for reftable (12 patches) landed with fuzzing infrastructure. The tone was collaborative but brisk, with maintainers pushing to finalize 2.55 follow-ups before the next cycle.

---

## Notable threads

### Git 2.55.0 released
**Junio C Hamano** announced the final release of Git 2.55.0, summarizing 505 non-merge commits from 100 contributors. Highlights include:
- **Parallel hooks** (`hook.jobs`, `--jobs`) and new built-ins (`git format-rev`, `git url-parse`).
- **Linux fsmonitor daemon** (Paul Tarjan) and **Rust support enabled by default** (Ezekiel Newren).
- **ODB abstraction** (Patrick Steinhardt) and **MIDX incremental repacking** (`--write-midx=incremental`).
- **Breaking changes**: Stricter proxy URL validation, sideband terminal control sequences disabled by default.

The release also includes over 50 bug fixes, from memory leaks to edge cases in `git bisect` and `git describe`. The changelog is organized by UI/workflow, performance, and internal refactoring, with notable contributions from Taylor Blau (MIDX), Jeff King (reftable leak fixes), and Hannes Sixt (new `git history` subcommands).

**Why it matters**: This is a major release with architectural shifts (ODB, Rust) and performance improvements (revision traversal, sparse-index). The parallel hooks feature is particularly significant for CI/CD workflows, while the ODB abstraction paves the way for pluggable backends.

---

### Reftable security hardening (12 patches)
**Patrick Steinhardt** posted a **v2 series** hardening Git’s reftable backend against maliciously corrupted files. The patches address vulnerabilities discovered via libFuzzer, including out-of-bounds reads/writes, NULL pointer dereferences, and uninitialized memory usage. Key fixes:
- **Patch 6/12**: Fixes a heap-buffer-overflow in log block parsing when the inflated block size is smaller than the header.
- **Patch 12/12**: Adds bounds checks in `reftable_table_new()` to prevent underflow when a file is truncated mid-footer.
- **Fuzzing infrastructure**: New Meson support for libFuzzer and a reftable fuzzer target (`oss-fuzz/fuzz-reftable.c`).

The series is **technically complete** and includes a test helper (`cl_reftable_write_block`) to reduce boilerplate in unit tests. No objections have been raised, and the fuzzer now runs for 2+ hours without surfacing new issues.

**Why it matters**: Reftable is a critical backend for large repositories (e.g., Google’s internal Git). This hardening effort proactively addresses security risks, even though exploitation requires local disk access. The fuzzing infrastructure will help prevent regressions.

---

### ODB abstraction: `struct object_info` refactoring
**Patrick Steinhardt** posted a **6-patch series** replacing `struct object_info`'s `whence` field with a new `struct object_info_source` to enable multi-source object resolution. The series is part of the ongoing ODB abstraction effort and has **conceptual approval** from Junio ("Great"), but a **medium-weight architectural debate** emerged over whether the `source` parameter should be opt-in (current design) or recorded centrally during `packed_git` initialization.

**Justin Tobler** and **Junio** questioned the opt-in design, suggesting it complicates the API. Patrick defended the approach as more flexible for callers that don’t need source information. The discussion remains unresolved, but the series is otherwise ready for review.

**Why it matters**: This refactoring is foundational for pluggable ODB backends. The debate highlights a tension between flexibility (opt-in) and simplicity (centralized tracking), which will likely resurface in future ODB work.

---

### `git replay --linearize` regression post-merge
**Johannes Schindelin** identified a **critical regression** in the recently merged `git replay --linearize` feature (Toon Claes). The bug causes **silent commit dropping** in single-branch replay with merge commits. For example, in a diamond-shaped history, only the tip commit is replayed, and intermediate commits (including merges) are lost.

**Patrick Steinhardt** raised a **CLI design inconsistency**: `--linearize` deviates from `git rebase`'s established syntax (`--rebase-merges`, `--no-rebase-merges`). The thread now centers on whether to adopt `git rebase`'s vocabulary for consistency.

**Why it matters**: This is a **high-priority regression** with data loss potential. The CLI debate is strategic—commands manipulating commit history should share vocabulary to avoid user confusion.

---

### `git history drop` and `git history squash` subcommands
**Hannes Sixt** posted two new subcommands for the experimental `git history` built-in:
- **`git history drop`** (11 patches): Removes a commit and replays its descendants onto its parent, with dry-run mode and refactoring of the reset machinery.
- **`git history squash`** (4 patches): Folds a commit range into its oldest commit, replaying descendants on top.

Both series are **technically complete** and address prior feedback, including input validation, merge commit handling, and `--reedit-message` usability. **Junio** and **Phillip Wood** reviewed the squash series, focusing on template formatting and edge cases (e.g., `fixup!` commits with out-of-range targets).

**Why it matters**: These subcommands modernize Git’s history-editing workflows, offering alternatives to `git rebase -i` with better conflict handling and dry-run support. The `drop` subcommand is particularly useful for removing sensitive commits without rewriting unrelated history.

---

## In brief

**`git refs` subcommands** -- Patrick Steinhardt’s 5-patch series adding `create`, `delete`, `update`, and `rename` subcommands to `git refs` was merged. The series consolidates reference manipulation functionality previously scattered across `git-update-ref` and `git-symbolic-ref`.

**ODB transactions in `receive-pack`** -- Justin Tobler’s 6-patch series refactoring `git-receive-pack` to use ODB transactions (instead of `tmp_objdir`) is ready for review. The series advances the ODB abstraction effort and includes a follow-up plan for backend-agnostic object writes.

**`paint_down_to_common()` optimization** -- Kristofer Karlsson’s 8-patch series optimizing merge-base computation for one-sided histories was merged. The series removes an obsolete commit-date fallback and adds a regression test.

**`USE_NSEC` debate** -- A discussion about flipping the default of `USE_NSEC` (nanosecond timestamp tracking) to `true` for most users gained traction. **Brian M. Carlson** proposed the change based on Jeff King’s testing, while **Patrick Steinhardt** suggested converting it to a runtime setting. The debate remains unresolved.

**`excludes_file` libification** -- Tian Yuchen’s series migrating `excludes_file` into `struct repo_config_values` is blocked on a **three-phase guardrail plan** (silent return → `BUG()` → no check). Junio insists **Phase 2 (adding `BUG()`)** must be implemented before merging.

**`greplint.pl` series** -- Michael Montalbo’s 6-patch series introducing a linter for `grep` assertions in tests was resolved. The series exposed systemic risks in automated conversion masking pre-existing test bugs, but the maintainer accepted it with targeted cleanup.

---

## On the radar

**`git replay --linearize` regression** -- The post-merge regression in `git replay --linearize` (silent commit dropping) needs urgent attention. A follow-up patch is expected to restore the `replayed_base` logic or redesign multi-branch handling.

**`git history` subcommands** -- The `drop` and `squash` subcommands are ready for final review. Junio’s "What’s cooking" report lists them as needing attention.

**Reftable hardening** -- Patrick Steinhardt’s 12-patch security-hardening series for reftable is technically complete and awaiting substantive review.

**ODB abstraction** -- The `struct object_info` refactoring series is stuck on the opt-in vs. centralized source-tracking debate. A resolution is needed to unblock further ODB work.