Here’s the digest for **2026/07/11**, a Saturday with moderate traffic (61 emails across 14 threads). The day was marked by late-stage design debates, procedural clarifications, and a flurry of documentation updates from Junio.

---

### The day in brief
A **technically dense but procedurally quiet** day. The most consequential activity was **Junio’s six-patch documentation series** clarifying contributor guidance, which codifies long-standing practices around commit messages, trailers, and the `seen` branch. Elsewhere, **Pablo Sabater’s `git log --graph` indentation series** hit a usability snag with Mirko Faina’s **bounded zig-zag counter-proposal**, while **Harald Nordgren’s `--delete-merged` series** inched closer to merge-readiness. A **performance optimization for `paint_down_to_common()`** (Tian Yuchen) is now unblocked after a procedural rebase, and **Patrick Steinhardt’s ODB abstraction series** saw follow-up cleanup from Jeff King.

---

### Notable threads

#### **`git log --graph` cascading indentation: usability redesign debate**
**Headline:** Mirko Faina’s **bounded zig-zag counter-proposal** forces a late-stage pivot in Pablo Sabater’s v9 series.
**What’s happening:** The series, which adds cascading indentation to disambiguate visual roots in `git log --graph`, is technically complete (lookahead buffer, edge-case fixes, test coverage) and has Junio’s architectural approval. However, Mirko’s review raised a **user-experience objection**: always-on indentation clutters output for simple cases (e.g., sequential one-patch series). His counter-proposal—a **bounded zig-zag pattern** (capping indentation at two levels before resetting) with opt-in behavior (triggered by `--oneline` or `--format=reference`)—offers a middle ground. Pablo is open to alternatives but defends the current opt-out default (`--no-graph-indent`).
**Why it matters:** This is a **late-stage inflection point** for a series that’s been through nine revisions. The maintainers must now rule on whether the usability trade-off is acceptable as-is or requires a follow-up patch. The debate highlights the tension between **correctness** (fixing ambiguity) and **readability** (avoiding clutter).
**Next steps:** Awaiting Junio’s ruling on the zig-zag proposal or a decision to defer the redesign.

---

#### **`git branch --delete-merged`: final test refinements**
**Headline:** Harald Nordgren’s v18 series is **implementation-complete and ready for merging** after addressing Phillip Wood’s test feedback.
**What’s happening:** The series adds `--delete-merged` to `git branch` with **refined stacked-branch protection** (abort-and-clear), per-branch opt-out (`branch.<name>.deleteMerged=false`), and `--dry-run` preview. Today’s updates focused on **test suite polish**: clearer branch names (`local-to-delete` instead of `on-local`), explicit `branch.main.pushRemote` configuration to prevent misclassification, and redundant command removal. The only remaining action is a **mechanical flags cleanup** (acknowledged by Harald) to resolve a maintenance risk in patches 2-4/7.
**Why it matters:** This is a **long-awaited safety feature** for local branch cleanup, with **18 revisions** reflecting the project’s rigorous review process. The stacked-branch protection (abort-and-clear) is particularly notable for its handling of dependency chains (e.g., `b1→b2→b3`).
**Next steps:** Junio to queue the series once the flags cleanup is applied.

---

#### **`paint_down_to_common()` optimization: procedural unblocking**
**Headline:** Tian Yuchen’s v6 series is **ready to merge** after Kristofer Karlsson clarified the procedural base.
**What’s happening:** The series optimizes `paint_down_to_common()` for one-sided histories, delivering **100-1000x speedups** for asymmetric merge-base queries (e.g., repositories with import grafts). Junio initially rejected the rebase on `next` due to unrelated topics, but Kristofer confirmed the series **applies cleanly to a synthetic base** (stable tag merged with `kk/commit-reach-find-all-fix`). The only remaining action is for Tian to **resubmit v6 with an updated cover letter** clarifying the base.
**Why it matters:** This is a **low-risk, high-reward optimization** addressing a long-standing performance bottleneck. The series is **fully reviewed** (Derrick Stolee, SZEDER Gábor) and includes **deterministic Trace2 instrumentation** to validate the speedups.
**Next steps:** Tian to resubmit; Junio to queue in `seen`.

---

#### **ODB abstraction: follow-up cleanup and design questions**
**Headline:** Jeff King’s cleanup patch (removing unused `add_object_entry()` return) is **ready to proceed**, while Taylor Blau and Peff debate traversal API consistency.
**What’s happening:** Patrick Steinhardt’s v2 ODB abstraction series (introducing object filters to `odb_for_each_object()`) saw two developments:
1. **Cleanup patch:** Peff submitted a follow-up to remove the unused `int` return from `add_object_entry()`, which Junio endorsed. The patch is **ready to merge** and could land independently or alongside the ODB series.
2. **Design question:** Peff and Taylor discussed whether `traverse_bitmap_commit_list()` and `traverse_commit_list()` should remain consistent in their handling of callback return values. The former now propagates non-zero returns for early termination, while the latter ignores them (callbacks return `void`). Peff suggests standardizing the APIs, but this is **non-blocking** for the series.
**Why it matters:** The ODB abstraction effort is foundational for pluggable backends, and the cleanup patch eliminates a long-standing inconsistency. The traversal API debate, while technical, could affect future callers.
**Next steps:** Junio to queue the cleanup patch; Patrick to address the traversal API question in a follow-up.

---

#### **Documentation: contributor guidance clarified**
**Headline:** Junio’s six-patch series **codifies existing practices** around commit messages, trailers, and the `seen` branch.
**What’s happening:** The series updates `Documentation/MyFirstContribution.adoc` and `Documentation/SubmittingPatches` to address recurring questions:
- **Commit message structure:** Formalized as **Observation** (problem), **Solution** (approach), **Command** (imperative instruction).
- **Trailer carry-over:** `Acked-by:` may be retained for minor changes; `Reviewed-by:` should not unless the patch is unchanged.
- **`seen` branch:** Explicitly described as an early integration step, not a signal of acceptance. "Will queue" in maintainer replies is clarified as a promise to add to `seen`, not to merge.
- **"What's cooking" style:** Past/present perfect tense in passive voice (e.g., "The XYZ subsystem has been updated to...").
**Why it matters:** These updates **reduce ambiguity for newcomers** and align documentation with long-standing project norms. The series is **uncontroversial** and likely to be merged as-is.
**Next steps:** Awaiting community feedback; no blockers identified.

---

### In brief
- **`git-subtree` safeguard:** Ian Jackson’s series (preventing mixing old and new implementations) is **awaiting v2** with test style improvements. Colin Stagner confirmed the current version is correct.
- **`USE_NSEC` discussion:** Junio withdrew the original patch, shifting focus to a **runtime-configurable approach** (`core.useNsec`). The build-time knob (`USE_NSEC`) will be deprecated.
- **`repo_config_values` migration:** Tian Yuchen’s v9 series (migrating 9 config variables) saw **design debate** with Pablo Sabater over submodule handling in `repo_config_values_clear()`. Tian agreed to align signatures but rejected direct struct access.
- **Test modernization:** Marcelo Machado Lage’s v2 series (modernizing `t9811-git-p4-label-import.sh`) is **under review** with no objections.
- **Double-free fix:** Patrick Steinhardt’s bugfix for `object-file.c` is **ready to merge**; Peff and Patrick clarified its non-security impact.
- **`git add -e` refactoring:** Gatla Vishweshwar Reddy’s v4 patch (replacing subprocess call with internal API) is **ready for final review** after addressing subdirectory correctness.
- **`fetch.submoduleErrors` test fixup:** Ramsay Jones’s patch (replacing `grep` with `test_grep`) is **awaiting integration** into the cooking topic.

---

### On the radar
- **`git log --graph` usability redesign:** The zig-zag counter-proposal could force a v10 reroll.
- **`repo_config_values` submodule handling:** Pablo Sabater’s concerns about silent leaks may resurface in a future iteration.
- **Traversal API consistency:** Peff and Taylor’s debate could lead to follow-up work on `traverse_commit_list()`.