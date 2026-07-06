Here’s the daily digest for **June 30, 2026**, covering the Git mailing list’s activity:

---

### **The day in brief**
A busy day with **94 emails across 23 threads**, dominated by **refactoring efforts** (ODB abstraction, repository setup, reftable hardening) and **post-merge follow-ups** (e.g., `git replay --linearize`’s regression and CLI debate). Key milestones: Patrick Steinhardt’s **ODB transaction series merged**, Phillip Wood’s **11-patch rebase-dropped-commit fix** posted, and Junio’s **“What’s cooking” report** signaling deep freeze for Git 2.55-rc2. Controversy lingers over `git replay --linearize`’s interface design, while usability discussions (e.g., `git blame -b` output) show the project’s responsiveness to edge-case friction.

---

### **Notable threads**

#### **ODB abstraction stack lands in `next`**
**Topic**: `ps/odb-source-packed` and related series
**Author**: Patrick Steinhardt
**Status**: Merged to `next` (Junio’s “What’s cooking” report)
**Summary**: The ODB abstraction effort reached a milestone with the merge of `ps/odb-source-packed` (18 commits), which refactors the packed object source into a proper `struct odb_source`. Dependent series (`ps/odb-drop-whence`, `ps/connected-generic-promisor-checks`, `ps/odb-generalize-prepare`) are now unblocked, with `ps/odb-generalize-prepare` approved by Toon Claes and Junio after addressing review feedback. The work enables pluggable ODB backends and is foundational for future features like `git grep` with non-files backends.
**Why it matters**: This is the most architecturally significant effort in flight, with implications for performance (e.g., reftable backend optimizations) and extensibility (e.g., remote object info). The merge signals confidence in the design, though follow-up work (e.g., backend-agnostic object writes) remains.

---

#### **Rebase-dropped-commit bug fixed systemically**
**Topic**: `[PATCH 0/11] sequencer: avoid copying notes from dropped commits`
**Author**: Phillip Wood
**Status**: Posted (v1), awaiting review
**Summary**: Phillip Wood delivered a **11-patch series** fixing the long-standing bug where `git rebase` incorrectly copies notes from dropped commits to HEAD. The series refactors the sequencer’s control flow, introduces an `enum pick_result` for clarity, and adds a `PICK_RESULT_DROPPED` state to avoid recording dropped commits as rewritten. It also fixes three additional rebase bugs (external merge strategy failures, command execution failures, and final fixup cleanup) and restores test coverage for the "apply" backend.
**Why it matters**: This is a **tested-level contribution** that addresses technical debt in the rebase machinery, improving correctness for both users and tools (e.g., post-rewrite hooks). The series is well-structured, with each patch building logically on the last, and is likely to land soon given Phillip’s track record and the bug’s severity.

---

#### **`git replay --linearize`’s regression and CLI debate**
**Topic**: `tc/replay-linearize` (v5 merged, but critical issues remain)
**Authors**: Toon Claes, Johannes Schindelin, Patrick Steinhardt
**Status**: Post-merge follow-up (regression + design debate)
**Summary**: The `git replay --linearize` feature (merged to `master` on July 15) has two critical issues:
1. **Silent commit dropping**: The base-selection logic in v5 inadvertently removed the `replayed_base` mechanism, causing only the tip commit to be replayed when squashing a single branch with merges (e.g., `master~2..master`).
2. **CLI inconsistency**: The `--linearize` flag deviates from `git rebase`’s `--rebase-merges` syntax, sparking a debate between **usability** (Schindelin: “`--linearize` is clearer”) and **consistency** (Steinhardt: “mirror `git rebase`’s interface”).
Toon Claes acknowledged the regression and agreed to restore `replayed_base`, but the CLI debate remains unresolved. Phillip Wood later proposed making `--reedit-message` the default, framing it as a **commit hygiene issue**.
**Why it matters**: This thread highlights the tension between **innovation** and **consistency** in Git’s CLI design. The regression is a high-priority fix, while the interface debate could shape future commands that manipulate commit history. Expect follow-up patches to address both issues before Git 2.55.

---

#### **`git history squash`’s usability feedback**
**Topic**: `hn/history-squash` (v6 merged)
**Authors**: Harald Nordgren, Matt Hunter, Phillip Wood
**Status**: Post-merge follow-up (user feedback)
**Summary**: Harald Nordgren’s `git history squash` feature (merged in v6) received **user feedback** from Matt Hunter, who raised three points:
1. **Template clutter**: The `--reedit-message` template includes fully commented-out `fixup!` messages, which Matt and Phillip Wood agreed should be omitted (retaining only a summary list of subjects).
2. **`squash!` vs. `fixup!`**: Matt asked whether `squash!` messages should be treated differently from `fixup!` (retained vs. discarded). Phillip Wood proposed a template that preserves `squash!` bodies while discarding `fixup!` content.
3. **Default behavior**: Matt questioned why `--reedit-message` isn’t the default, suggesting it encourages better commit hygiene. Phillip Wood endorsed this, arguing for interactivity by default.
Harald later deferred template changes to maintain consistency with `git rebase -i`, but the discussion surfaced broader questions about **recoverability** (e.g., undoing multi-ref operations) and **workflow expectations**.
**Why it matters**: This thread shows how **user feedback** can refine even merged features. The template debate is minor, but the default behavior question touches on Git’s philosophy: **speed vs. safety**. Expect follow-up patches to adjust the template or document the trade-offs.

---

#### **Reftable hardening and fuzzing infrastructure**
**Topic**: `ps/reftable-hardening` (v2, 12 patches)
**Author**: Patrick Steinhardt
**Status**: Ready for review (Junio’s “What’s cooking” report)
**Summary**: Patrick Steinhardt’s **12-patch series** hardens the reftable backend against corrupted files, fixing OOB reads/writes, NULL pointer dereferences, and `abort()` calls discovered via libFuzzer. The series also adds **fuzzing infrastructure** (Meson build support for libFuzzer) and a test helper (`cl_reftable_write_block`) to reduce boilerplate in unit tests. Junio acknowledged the test helper in a brief reply, signaling no objections.
**Why it matters**: This is the first time Git’s build system has been extended to support **coverage-guided fuzzing**, a significant step for security hardening. The fixes are comprehensive, and the fuzzer will help prevent regressions. The series is likely to land soon, given its self-contained nature and lack of controversy.

---

#### **`git blame -b`’s output formatting fixed**
**Topic**: `git blame -b` output formatting
**Authors**: Laszlo Ersek, Junio C Hamano, René Scharfe
**Status**: Patch posted (René Scharfe)
**Summary**: Laszlo Ersek reported that `git blame -b` reserves an extra hex digit for a caret marker that is never displayed, causing commit hashes to be one nibble longer than `core.abbrev`. Junio initially suggested the issue lay in `git rebase`’s parser, but Laszlo clarified it was a **manual workflow friction** (e.g., pasting hashes into `git rebase -i`). René Scharfe posted a **patch** that refactors the mark-handling logic to count and print marks only when they’re shown, aligning the hash length with `core.abbrev`.
**Why it matters**: This is a **small but impactful usability fix** that eliminates a persistent annoyance for users who rely on `git blame -b` for interactive rebases. The patch is minimal and well-motivated, with no backward-compatibility risks identified yet.

---

### **In brief**
- **`git refs` subcommands merged**: Patrick Steinhardt’s series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` landed in `next` after a final typo fix. The work consolidates reference manipulation under a unified interface.
- **Memory leaks plugged**: Jeff King fixed a leak in `git format-patch`’s `prepare_bases()` and adjusted the test harness to redirect LSan output to stderr, improving CI reliability.
- **Repository setup refactored**: Patrick Steinhardt’s **13-patch series** to separate repository discovery from setup (introducing `struct repo_discovery`) posted, with Junio fixing a typo in one commit. The work is foundational for future setup logic consolidation.
- **Config case-sensitivity fixed**: Rishav Dewan’s patch fixed a long-standing bug where `git config` failed to update values in old-style `[section.subsection]` headers if the subsection contained uppercase letters. Junio redirected the review to Johannes Schindelin.
- **Rust build dependency removed**: Jan Palus removed a spurious dependency in the Makefile where the Rust static library (`$(RUST_LIB)`) incorrectly depended on the C library (`$(LIB_FILE)`), reducing unnecessary rebuilds.
- **`git history reword` leak fixed**: Junio C Hamano’s v2 patch fixed a file stream leak in `git history --reword` and restored error checking on `fwrite()`, addressing Patrick Steinhardt’s review feedback.
- **Test modernization**: Bryan B. Lima updated `t7412-submodule-absorbgitdirs.sh` to use descriptive test helpers (`test_path_is_file`, etc.), improving test failure messages.

---

### **On the radar**
- **`git replay --linearize` follow-ups**: Expect patches to fix the regression (restoring `replayed_base`) and possibly adjust the CLI interface (e.g., adopting `--rebase-merges` syntax or making `--reedit-message` the default).
- **`git history squash` template changes**: Phillip Wood’s proposal to clean up the `--reedit-message` template (omitting `fixup!` noise) may land in a follow-up patch.
- **ODB abstraction follow-ups**: Justin Tobler’s series converting `git-receive-pack` to use ODB transactions is merged, but backend-agnostic object writes remain a future goal.
- **`USE_NSEC` debate**: The discussion about flipping the default or converting it to a runtime setting continues, with Jeff King noting the lack of a portable auto-detection mechanism.
- **Sparse-checkout in `git clone`**: Pushkar Singh’s RFC for `--only`/`--except` options received feedback from Jeff King, who suggested a patterns-file approach. The feature’s usefulness remains unproven.

---
**Volume**: Heavy (94 emails), but focused on **refactoring** and **post-merge follow-ups**. The day’s most consequential work was Phillip Wood’s **rebase-dropped-commit fix** and Patrick Steinhardt’s **ODB abstraction stack** merging to `next`. Controversy lingers over `git replay --linearize`’s interface, while usability discussions (e.g., `git blame -b`) show the project’s responsiveness to edge cases.