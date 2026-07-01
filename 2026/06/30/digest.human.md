Here’s the daily digest for **June 30, 2026**, covering the Git mailing list traffic:

---

### **The day in brief**
A busy but focused day on the Git mailing list, with **94 emails across 23 threads**. The tone was **collaborative and milestone-heavy**, with several long-running efforts reaching resolution or advancing toward integration. **Notable highlights**:
- **Phillip Wood’s 11-patch series fixing rebase-dropped-commit bugs** (including notes corruption) was **merged after Junio’s final sign-off**.
- **Patrick Steinhardt’s ODB abstraction work** saw key progress, with `ps/odb-generalize-prepare` **approved for `next`** and `ps/odb-source-packed` already merged.
- **Harald Nordgren’s `git history squash`** feature reached consensus on usability tweaks, with Junio’s final review pending.
- **A memory leak in `git format-patch`** was fixed by Jeff King, alongside a test harness improvement for LeakSanitizer.
- **René Scharfe posted a patch** to fix `git blame -b`’s inconsistent hash abbreviation, addressing a long-standing usability friction.

The day’s texture was **dominated by refactoring and bugfixes**, with a few philosophical debates (e.g., CLI consistency in `git replay`) and usability discussions (e.g., reflog design) simmering in the background.

---

### **Notable threads**

#### **Rebase-dropped-commit bugs fixed**
**Subject**: [PATCH v3 0/11] sequencer: avoid copying notes from dropped commits
**Author**: Phillip Wood
**Status**: **Merged** after Junio’s final sign-off.
**Summary**: An 11-patch series that **systemically fixes bugs in Git’s rebase machinery** where dropped commits (due to empty changes, fixup sequences, or merge failures) incorrectly copied their notes to the current HEAD or triggered post-rewrite hooks. The series introduces `PICK_RESULT_DROPPED` to the sequencer’s result enum and refactors control flow to skip `record_in_rewritten()` for dropped commits. **Key fixes**:
- Silent notes corruption in single-branch replay.
- Post-rewrite hook inaccuracies for fixups + empty changes.
- Edge cases in external merge strategy failures and command execution failures (e.g., `git commit` failing to run).
**Test coverage**: Expanded in `t3400-rebase.sh` and `t5407-post-rewrite-hook.sh`.
**Why it matters**: This resolves a **long-standing correctness issue** that could silently corrupt repository state, particularly for users relying on notes or hooks. The series is a **testament to Phillip’s deep engagement with the rebase subsystem**, addressing not just the reported bug but also systemic technical debt.

---

#### **ODB abstraction advances**
**Subject**: [PATCH v1 0/2] odb: generalize reprepare() callback
**Author**: Patrick Steinhardt
**Status**: **Approved for `next`** after Toon Claes’s substantive review and Junio’s explicit approval.
**Summary**: A 3-patch series that **generalizes the `reprepare()` callback** into a more flexible `prepare()` callback with an `ODB_PREPARE_FLUSH_CACHES` flag, enabling `git grep` to work with pluggable ODB backends. The series replaces direct calls to backend-specific functions (e.g., `odb_source_packed_prepare()`) with a generic `odb_source_prepare()` that delegates through the vtable, and introduces `odb_prepare()` to pre-open ODB sources.
**Key changes**:
- New `enum odb_prepare_flags` with `ODB_PREPARE_FLUSH_CACHES`.
- `odb_transaction_begin()` now takes flags and disallows `NULL` output on success.
- Backward compatibility preserved via `odb_reprepare()` wrapper.
**Why it matters**: This is a **critical step toward pluggable ODB backends**, a long-term goal for Git’s architecture. The series is **well-scoped and uncontroversial**, with all prior concerns (e.g., downcasting, flag semantics) resolved.

---

#### **`git history squash` usability finalized**
**Subject**: [PATCH v6 0/4] history: add `squash` subcommand
**Author**: Harald Nordgren
**Status**: **Ready for Junio’s final review** after addressing all feedback, including usability tweaks to the `--reedit-message` template.
**Summary**: A 4-patch series adding `git history squash` to fold a range of commits into one while replaying descendants. **Key design points resolved**:
- **Merge commit handling**: Rejects merges with external parents but allows fully contained merges.
- **Ref-handling**: Rejects operations with interior refs by default, with advice to use `--update-refs[=head]`.
- **`--reedit-message` template**: Adopts Phillip Wood’s finalized layout (summary list of subjects, grouped `fixup!`/`amend!` commits, minimal separators).
- **Recoverability**: Matt Hunter and Phillip Wood highlighted that `git reset --hard` is insufficient for undoing `--update-refs` operations, but this was deemed a **systemic issue** (not specific to `squash`) and deferred.
**Why it matters**: This feature **fills a gap in Git’s history-editing toolkit**, offering a more efficient alternative to `git rebase -i` for collapsing ranges. The series is **code-complete and uncontroversial**, with only minor usability tweaks remaining.

---

#### **Memory leaks plugged**
**Subject**: [PATCH 0/2] fix memory leaks in format-patch and test harness
**Author**: Jeff King
**Status**: **Merged**.
**Summary**: Two patches fixing:
1. A **leak in `git format-patch`** (since 2016) where a `rev_info` struct allocated in `prepare_bases()` was never freed. Triggered by commit-graph support (`GIT_TEST_COMMIT_GRAPH=1`).
2. A **test harness annoyance** where LeakSanitizer (LSan) output was misdirected to stdout, causing TAP parse errors with `prove`.
**Why it matters**: The `format-patch` fix is a **long-overdue cleanup**, while the test harness improvement **reduces friction for developers** running leak tests. Patrick Steinhardt proposed enabling LSan for the `linux-TEST-vars` CI job to catch similar leaks earlier.

---

#### **`git blame -b` usability fix**
**Subject**: [PATCH] blame: stop reserving extra hex digit for unused caret marker
**Author**: René Scharfe
**Status**: **Posted**, awaiting review.
**Summary**: Fixes a **long-standing usability friction** in `git blame -b` where non-boundary commit hashes were abbreviated to `core.abbrev + 1` hex chars to reserve space for an unused caret marker. The patch refactors mark-handling logic to count and print marks only when they’re actually shown, aligning hash lengths with `core.abbrev`.
**Why it matters**: This **eliminates manual truncation** when pasting hashes from `git blame -b` into `git rebase -i`, a common workflow for users. The patch is **minimal and well-motivated**, though it lacks test coverage.

---

### **In brief**
- **`git refs` subcommands merged**: Patrick Steinhardt’s 5-patch series adding `delete`, `update`, `create`, and `rename` to `git refs` was **merged after a final typo fix**. This consolidates reference manipulation under a unified interface.
- **Reftable hardening**: Patrick Steinhardt’s 12-patch series fixing OOB reads/writes and NULL pointer dereferences in the reftable backend was **acknowledged by Junio** (test helper improvement). The series adds a libFuzzer-based fuzzer to prevent regressions.
- **`USE_NSEC` debate**: Jeff King’s testing showed modern Linux filesystems preserve nanosecond timestamps, obsoleting historical "racy Git" concerns. The discussion now centers on **runtime configuration** (e.g., `core.useNsec`) vs. build-time knobs, with no consensus yet.
- **`git replay --linearize` CLI debate**: Johannes Schindelin and Patrick Steinhardt clashed over **UX vs. consistency**, with Patrick conceding divergence from `git rebase`’s syntax but demanding **explicit justification in commit messages**. The regression in the merged v5 series (silent commit dropping) remains unaddressed.
- **Test modernization**: Bryan B. Lima’s patch replacing raw test calls in `t7412-submodule-absorbgitdirs.sh` with descriptive helpers was **queued by Junio** as "perfect" for a first-time contributor.

---

### **On the radar**
- **`git replay --linearize` regression**: The merged v5 series has a **critical bug** where single-branch replay silently drops commits. Toon Claes clarified the intended design (single linear sequence regardless of input branches), but a **follow-up patch is urgently needed**.
- **Reftable performance**: Jeff King identified **quadratic-time behavior** in the reftable backend during bulk ref deletion/re-creation, a more impactful issue than the earlier `stat()` overhead. No patch yet, but this could inform future optimizations.
- **CI visibility**: Junio accepted Patrick Steinhardt’s proposal to add a **GitLab CI status badge** to `README.md`, addressing a maintainer visibility concern. The third patch (enabling `GIT_TEST_LONG` for GitLab CI) remains on hold.
- **`git clone --only/--except`**: Pushkar Singh’s RFC for sparse-checkout paths during clone received **cautious feedback from Jeff King**, who suggested a file-based patterns approach. No implementation yet.

---