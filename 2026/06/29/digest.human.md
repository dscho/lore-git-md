## The day in brief

**Monday, June 29, 2026** was a **heavy, milestone-heavy day** on the Git mailing list. The **Git 2.55.0 release** dominated the traffic, but beneath that headline, the list buzzed with **architectural debates, security hardening, and long-running series reaching critical junctures**. The tone was **pragmatic and forward-looking**—a mix of wrapping up loose ends before the release and setting the stage for the next cycle.

### What you absolutely should not miss:

- **Git 2.55.0 shipped**, with **parallel hooks, Rust enabled by default, and a native Linux fsmonitor daemon** as standout features.
- **`git history drop` (Patrick Steinhardt) and `git history squash` (Harald Nordgren) both reached v7**, with the former now **technically complete** and the latter **awaiting final polish** on its `--reedit-message` template.
- **The `git log --graph` cascading indentation series (Pablo Sabater) hit an architectural crossroads**, with a **lookahead buffer redesign** now favored over the v6 peek-based abstraction.
- **The reftable backend hardening series (Patrick Steinhardt) landed v2**, adding **fuzzing infrastructure and 10 security fixes** for corrupted files.

---

## Notable threads

### `git log --graph` cascading indentation – architectural crossroads
**Headline:** *Kristofer Karlsson’s lookahead buffer redesign now favored over v6 peek-based abstraction*
**What’s happening:** Pablo Sabater’s series to add cascading indentation for visual roots in `git log --graph` has reached a **critical decision point**. Junio Hamano signaled readiness to adopt Kristofer’s **lookahead buffer redesign**, which resolves the three failing test cases and eliminates the fragility of peeking at unsimplified commits. The redesign removes the need for the two new peek functions entirely, simplifying the interface between `graph.c` and `revision.c`. Pablo confirmed he is preparing a v7 reroll implementing this approach.
**Where it stands:** Architecturally settled; v7 expected shortly. The series is now a **clean, robust solution** rather than a fragile workaround.

---

### `git history drop` – v7 complete, one blocking review
**Headline:** *Patrick Steinhardt’s 11-patch series is technically complete, but Junio Hamano identified a logical flaw in `find_head_tree_change()`*
**What’s happening:** The `git history drop` subcommand, which removes a commit and replays its descendants onto its parent, reached v7 with all prior feedback addressed. The series refactors the reset machinery, adds a dry-run mode, and modernizes the API to avoid touching HEAD or the index unnecessarily. However, Junio’s review of the final patch surfaced a **logical flaw**: when `--update-refs=head` is used, `find_head_tree_change()` incorrectly concludes that HEAD does not move because it looks for the *symbolic name* of the current branch in an array that only contains `HEAD`. This could leave the repository in an inconsistent state.
**Where it stands:** **One blocking issue** remains; a v8 is expected. The series is otherwise **ready for `next`**, with thorough test coverage and clear commit messages.

---

### `git history squash` – v6 complete, minor usability polish
**Headline:** *Harald Nordgren’s series is code-complete, with only cosmetic `--reedit-message` template tweaks remaining*
**What’s happening:** The `git history squash` subcommand, which folds a range of commits into one and replays descendants on top, reached v6 with all major design questions resolved. The series now rejects operations with refs pointing to interior commits (with advice to use `--update-refs=head`), supports multiple revision arguments, and includes a `--reedit-message` flag that seeds the editor with all folded-in commit messages. Junio and Phillip Wood raised **minor usability concerns** about the template format (numbered markers vs. cleaner separation), but the core functionality is uncontested.
**Where it stands:** **Ready for final review** pending a v7 with cosmetic template tweaks. The series is **well-tested and well-documented**, with 550+ lines of test coverage.

---

### Reftable backend hardening – v2 with fuzzing infrastructure and 10 security fixes
**Headline:** *Patrick Steinhardt’s 12-patch series adds libFuzzer support and fixes OOB reads/writes, NULL derefs, and aborts on corrupted files*
**What’s happening:** The series systematically hardens the reftable backend against maliciously corrupted files, addressing vulnerabilities discovered via fuzzing. v2 adds **Meson build support for libFuzzer**, a new fuzzer target (`oss-fuzz/fuzz-reftable.c`), and **10 individual fixes** for edge cases in block parsing, restart offsets, and record initialization. Each fix is paired with a unit test, and the fuzzer now runs for 2+ hours without surfacing new issues. The series is part of the ongoing reftable effort led by Patrick and Karthik Nayak.
**Where it stands:** **Ready for substantive review**. The infrastructure and fixes are **comprehensive and proactive**, with no known regressions.

---

### ODB abstraction – `ps/odb-drop-whence` and `ps/odb-generalize-prepare`
**Headline:** *Patrick Steinhardt’s ODB refactoring series advance, with Justin Tobler and Junio Hamano raising architectural questions*
**What’s happening:** Two series in Patrick’s ODB abstraction effort are in flight:
- **`ps/odb-drop-whence` (7 patches)**: Replaces the `whence` field in `struct object_info` with an opt-in `struct object_info_source` to enable multi-source object resolution. Junio conceptually approved the direction ("Great"), but Justin Tobler and Junio raised **architectural questions** about where source-tracking logic should live (caller vs. backend initialization).
- **`ps/odb-generalize-prepare` (3 patches)**: Generalizes the `reprepare()` callback into a `prepare()` callback with a flush flag, enabling `git grep` to work with pluggable ODB backends. Junio’s review was brief but approving ("Are we all happy with the current shape?").
**Where it stands:** **Ready for `next`** pending resolution of the architectural questions in `ps/odb-drop-whence`. The series are **foundational for pluggable ODB backends** and well-scoped.

---

### `git replay --linearize` – design debate on interface consistency
**Headline:** *Patrick Steinhardt pushes back on Toon Claes’s incremental approach, advocating for `git rebase`-style `--rebase-merges` syntax*
**What’s happening:** Toon Claes’s `git replay --linearize` series (v5 merged) added a `--linearize` option to flatten merge commits into linear history. Patrick Steinhardt responded with a **design critique**, arguing that if the eventual goal is to support all of `git rebase`'s modes (`--no-rebase-merges`, `--rebase-merges`, `--rebase-merges=rebase-cousins`), the interface should adopt `git rebase`'s `--rebase-merges=<mode>` syntax from the start. The discussion is **forward-looking**—no code changes yet, but the interface choice could affect long-term consistency.
**Where it stands:** **Design debate open**; Toon is expected to respond with either a revised interface or a documented rationale for the current design.

---

### `paint_down_to_common()` optimization – v5 ready, one regression fix
**Headline:** *Tian Yuchen’s series is regression-free and fully reviewed, with a follow-up patch to remove the commit-date fallback*
**What’s happening:** The series optimizes `paint_down_to_common()` to terminate early when one side of a merge-base query exhausts its commit queue, yielding **100-1000x speedups** for asymmetric queries. v4 fixed a critical regression, and v5 is now **fully reviewed and ready to merge**. Kristofer Karlsson and Derrick Stolee identified a **new regression** affecting the `!FIND_ALL` early-exit optimization when the commit queue falls back to commit-date ordering (v1 commit-graph data). They agreed to **remove the fallback entirely**, as side-exhaustion renders it obsolete.
**Where it stands:** **Ready for `next`** pending a v5 reroll with the fallback removal. The series is **well-tested and well-documented**, with a new technical document (`paint-down-to-common.adoc`) explaining the algorithm.

---

### `USE_NSEC` – debate on defaults and runtime configuration
**Headline:** *Brian M. Carlson and Patrick Steinhardt propose flipping the default or converting to runtime configuration*
**What’s happening:** The `USE_NSEC` knob, which enables sub-second file timestamp tracking, is being re-evaluated in light of Jeff King’s testing showing it is redundant on modern Linux. Brian proposed **flipping the default to `true`** for most users, while Patrick suggested **converting it to a runtime-configurable setting** (like `core.ignoreCase`) to handle filesystem variability. Junio Hamano pushed back on treating it identically to `core.ignoreCase`, noting that `USE_NSEC` is optional even on capable systems.
**Where it stands:** **Debate open**; no consensus yet on whether to flip the default, deprecate the knob, or convert it to runtime control. The original Meson parity patch (adding the `nanosec` option) remains queued.

---

## In brief

- **`git refs` subcommands (Patrick Steinhardt)**: The series adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is **merged to `next`**. It consolidates reference manipulation functionality under a unified interface.
- **`git replay --linearize` regression (Toon Claes)**: Johannes Schindelin identified a **critical regression** in v5 where single-branch replay with merge commits silently drops commits. A follow-up patch is urgently needed.
- **Memoized commit traversal optimization (Tamir Duberstein)**: The series extending Git’s memoized commit-traversal optimization from `git tag --contains` to all ref-filter commands is **ready for review**. It adds robust cycle detection and trace2 instrumentation.
- **Shell completion dotfile hiding (Zakariyah Ali)**: The series to hide dotfiles by default in shell completion is **blocked on structural overhaul**. Junio nudged the author for a v4.
- **`trust_executable_bit` migration (Tian Yuchen)**: The series migrating `trust_executable_bit` into `repo_config_values` is **merged**, completing another step in the `the_repository` removal effort.
- **Priority queue optimization (Kristofer Karlsson)**: The series optimizing `prio_queue_get()` for one-sided histories is **approved and ready to merge**, with a hybrid approach combining sift-down and cascade optimizations.
- **`excludes_file` libification (Tian Yuchen)**: The series migrating `excludes_file` into `repo_config_values` is **ready for `next`**, with the guardrail debate resolved (centralizing initialization checks in `repo_config_values()`).
- **`git history --reword` bugfix (Junio Hamano)**: The patch fixing a file stream leak and inefficient file handling in `git history --reword` is **ready to merge** after addressing Patrick Steinhardt’s error-checking concern.
- **macOS CI hangs (Jeff King)**: The 2-patch series increasing Apache’s `Timeout` directive and isolating the expensive test case is **merged**. A follow-up patch to align GitLab CI coverage with GitHub’s is **on hold pending policy discussion**.
- **GSoC 2026 – partial clones (Siddharth Shrimali)**: The student’s progress report on improving disk space recovery for partial clones is **in the exploratory phase**, with no formal patches yet.
- **ODB transactions in `receive-pack` (Justin Tobler)**: The series refactoring `git-receive-pack` to use ODB transactions is **ready for v2**, with all review feedback addressed.
- **`git merge-base` clock-skew regression (Kristofer Karlsson)**: The 2-patch bugfix series is **ready to merge**, with Junio accepting Kristofer’s proposal to merge it as-is and rework the in-flight `kk/merge-base-exhaustion` topic later.

---

## On the radar

- **`git replay --linearize` interface consistency**: The debate between `--linearize` and `--rebase-merges=<mode>` could resurface if Toon Claes revisits the interface.
- **`USE_NSEC` default/runtime configuration**: The discussion about flipping the default or converting to runtime control is unresolved.
- **GitLab CI coverage alignment**: The policy question about whether GitLab CI should mirror GitHub’s coverage or run additional tests remains open.
- **`git history squash` template polish**: The `--reedit-message` template format debate is minor but could delay final review.