# Here is the daily digest for the Git mailing list on **2026/07/06 (Monday)**.

---

### **The day in brief**
A busy Monday with **109 emails across 35 threads**, dominated by **performance optimizations, refactoring, and usability improvements**. The standout developments:
- **`git log --graph` cascading indentation** (Pablo Sabater) reached **v7**, resolving architectural crossroads with a lookahead buffer and now **ready for final review**.
- **`git history squash`** (Harald Nordgren) landed in **v7**, adopting `git rebase -i`’s squash-message template and **marked "Will replace" by Junio**, signaling imminent merge.
- **`git refs` subcommands** (Patrick Steinhardt) are **fully merged**, consolidating reference manipulation under a unified interface.
- **Rebase dropped-commit notes** (Phillip Wood) saw **systemic overhaul v11 queued**, fixing long-standing leaks in the sequencer.
- **Reftable backend performance** (Patrick Steinhardt) received a **quadratic-time fix**, cutting runtime from 14s to 0.2s for bulk ref operations.
- **`GIT_TEST_LONG` tests** (Patrick Steinhardt) are now **reliable and CI-enabled** after a 9-patch series addressing hangs and inefficiencies.

The day’s tone was **collaborative and milestone-heavy**, with several long-running efforts nearing completion. No major controversies surfaced, though **Rustification build system adjustments** and **`git rebase -i -x` design debates** remain unresolved.

---

### **Notable threads**

#### **`git log --graph` cascading indentation (v7)**
**Headline**: Pablo Sabater’s **7-patch series** implementing cascading indentation for visual roots in `git log --graph` is now **architecturally resolved and ready for final review**. The v7 iteration replaces the fragile v6 peek-based abstraction with Kristofer Karlsson’s **lookahead buffer redesign**, addressing all failing test cases and earning maintainer approval for the approach.

### Key details

- **Core change**: A two-slot lookahead buffer in `struct git_graph` ensures commits are fully simplified before inspection, eliminating the need for peek functions (`revision_peek_next_commit()`).
- **Visualization**: Visual roots (except the first in a cascade) are indented by 2 chars per level while preserving parent-child relationships.
- **Test coverage**: 453-line test script (`t4218-log-graph-indentation.sh`) exercises edge cases (merges, `--first-parent`, `--left-right`, cascading sequences).
- **Compatibility**: The redesign aligns with Jeff King’s planned removal of `revs->commits` and Kristofer Haugsbakk’s `kk/prio-queue-get-put-fusion` topic.

**Status**: **Ready for integration** after minor cosmetic adjustments (e.g., `ARRAY_SIZE()` usage, assertions). Junio Hamano and Phillip Wood’s final reviews are pending.

---

#### **`git history squash` (v7)**
**Headline**: Harald Nordgren’s **5-patch series** introducing `git history squash` (folding a commit range into its oldest commit) reached **v7** and was **marked "Will replace" by Junio**, signaling intent to merge for the next release. The series now adopts `git rebase -i`’s squash-message template, resolving prior usability concerns.

### Key details

- **Behavior**: Rejects ranges whose oldest commit is a `fixup!`/`squash!`/`amend!` (target cannot be inside the range). By default, refuses operations where refs point to commits inside the squashed range, with advice to use `--update-refs=head`.
- **Template alignment**: `--reedit-message` now matches `git rebase -i`’s squash behavior, with `fixup!` messages fully commented out and `squash!`/`amend!` bodies retained.
- **Edge cases**: Explicitly rejects merges with external parents but allows fully contained merges.

### Open questions

- Phillip Wood’s review highlights **deviations from `rebase -i`**: the template does not reorder commits to place `fixup!`/`squash!` markers after their targets, and `amend!` messages are only partially commented.
- Whether `--reedit-message` (or `--edit`) should be the default remains unresolved.

**Status**: **Ready for merge** pending minor template tweaks.

---

#### **`git refs` subcommands (merged)**
**Headline**: Patrick Steinhardt’s **5-patch series** adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is **fully merged**, consolidating reference manipulation under a unified interface.

### Key details

- **Subcommands**:
  - `git refs delete`: Mirrors `git update-ref -d` with `--message` and `--no-deref` support.
  - `git refs update`: Atomic updates with `<old-value>` verification, deletion via `$ZERO_OID`.
  - `git refs create`: Explicit atomic creation (rejects existing refs, even with the same value).
  - `git refs rename`: Validates names, migrates reflogs, but **does not support symref renaming** (backend limitation).
- **Usability**: `--no-deref` controls symref handling (e.g., fails if target exists as a symref) but does not enable symref creation.

**Status**: **Merged to `master`**. Follow-up work may address symref renaming and reflog creation in `rename`.

---

#### **Rebase dropped-commit notes (v11 queued)**
**Headline**: Phillip Wood’s **11-patch systemic overhaul** fixing notes leaks in the sequencer is **queued for merging**. The series prevents notes from being copied from dropped commits during rebase, addressing a long-standing correctness issue.

### Key details

- **Core change**: Adds `PICK_RESULT_DROPPED` to `enum pick_result` and modifies `pick_one_commit()` to skip `record_in_rewritten()` for dropped commits.
- **Edge cases**: Handles empty changes, fixup sequences, edit commands, skip operations, **external merge strategy failures**, and **command execution failures** (e.g., `git commit` failing).
- **Refactoring**: Introduces `enum pick_result` and simplifies control flow in `sequencer.c`.

**Status**: **Queued in Junio’s tree** with minor typofixes. A known limitation (edit command still records dropped commits) is documented for a separate fix.

---

#### **Reftable backend performance fix**
**Headline**: Patrick Steinhardt’s **2-patch series** fixes a **quadratic-time regression** in the reftable backend when re-creating refs that were previously deleted (tombstoned). The fix moves tombstone handling from the merged iterator to call sites, enabling early termination and cutting runtime from **14s to 0.2s** for 8,000 refs.

### Key details

- **Root cause**: The merged iterator’s `suppress_deletions` flag forced a full scan of tombstones before bounds checks could terminate iteration.
- **Fix**: Removes `suppress_deletions` and handles tombstones explicitly at call sites (e.g., `refs_verify_refnames_available()`).
- **Test coverage**: New perf test (`p1401-ref-store-tombstones.sh`) and correctness test (`t0610-reftable-basics.sh`).

**Status**: **Ready for review**. No objections raised.

---

#### **`GIT_TEST_LONG` tests (v3)**
**Headline**: Patrick Steinhardt’s **9-patch series** makes `GIT_TEST_LONG` tests **reliable and efficient enough for CI**, addressing hangs and inefficiencies in GitHub and GitLab CI.

### Key changes

- **Test fixes**:
  - Skip `t0021` (2 GB clean/smudge) on 32-bit platforms (`SIZE_T_IS_64BIT` prerequisite).
  - Replace `dd ibs=1` with `genzeros` in `t4141`, reducing runtime from 6 minutes to <1 second.
  - Reduce peak disk usage in `t5608` and `t7900` via `test_when_finished rm -rf`.
- **CI changes**:
  - Enable `GIT_TEST_LONG` for GitLab CI pushes to integration branches (excluding Windows runners).
  - Remove macOS RAM-disk workaround (trading 7–12 minute runtime increase for stability).

**Status**: **Ready for merge**. Junio confirmed the interdiff (v2→v3) is "trivially correct."

---

### **In brief**
- **`git blame -b` output formatting**: René Scharfe’s patch fixing hash abbreviation alignment in `git blame -b` is **ready for integration** after Junio’s review.
- **`greplint.pl` (v4)**: Michael Montalbo’s **6-patch series** introducing a linter to convert bare `grep` to `test_grep` is **approved for merging**, with 10+ bugfixes and shared parser infrastructure.
- **`git subtree` safeguards**: Ian Jackson’s **2-patch series** adds detection for Rust rewrite output in the shell-script `git subtree`, preventing mixed-tool corruption.
- **Sparse-index segfault**: Derrick Stolee’s patch fixes a segfault when collapsing a full index with intent-to-add entries outside the sparse-checkout cone.
- **`git rm` error reporting**: A patch improving `git rm`’s error messages via `die_errno()` is **blocked** pending call-graph analysis to confirm `errno` provenance.
- **Trace2 ancestry test**: A patch fixes `t0213-trace2-ancestry.sh` failures in user-mode emulation (e.g., QEMU) by skipping tests when process names are unreliable.

---

### **On the radar**
- **`git rebase -i -x`**: Trevor Gross’s patch series faces **high-weight objections** from Junio, Peff, and Phillip Wood, who argue the feature misaligns with `rebase`’s core semantics. A pivot to `cherry-pick -i` is likely.
- **Rustification build system**: Shardul Natu and Koji Nakamaru’s **v6 series** for macOS Universal Binary support is **unblocked** after resolving duplicate `main()` symbol conflicts. Junio’s final review is pending.
- **`USE_NSEC` runtime configuration**: Patrick Steinhardt proposed making `USE_NSEC` runtime-configurable (e.g., `core.useNsec`) to avoid auto-detection challenges, but the default value and interoperability risks remain unresolved.
- **`git refs` usability**: Toon Claes’s feedback on `--no-deref`’s counterintuitive behavior in `git refs create` may inform future refinements.

---

### **Editorial notes**
The day’s traffic reflected **Git’s maturing infrastructure**, with several long-running efforts (e.g., `git log --graph`, `git refs`, reftable backend) reaching critical milestones. The **collaborative tone** was notable—even contentious threads (e.g., `git rebase -i -x`) saw reviewers proposing constructive alternatives rather than outright rejection. Performance optimizations (reftable, `GIT_TEST_LONG`) and **test suite modernization** (`greplint.pl`, `t1410-reflog.sh`) dominated, underscoring the project’s focus on **scalability and reliability**.

No major controversies emerged, though **Rustification** and **`USE_NSEC`** remain thorny topics with unresolved design questions. The **`git history squash`** series stands out as a rare example of a new user-facing feature gaining traction, thanks to its alignment with existing workflows (`git rebase -i`).