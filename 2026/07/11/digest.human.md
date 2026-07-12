Here is the digest for **2026/07/11 (Saturday)**, covering 61 emails across 14 active threads.

---

### The day in brief
A **moderately busy Saturday** on the Git mailing list, with **four threads poised for near-term resolution** and **two major design discussions**—one on `git log --graph` usability, the other on `USE_NSEC` deprecation—shaping future work. The **`paint_down_to_common()` optimization series** cleared its last procedural hurdle and is ready for `seen`, while **Harald Nordgren’s `--delete-merged` series** awaits only a mechanical flags cleanup. **Documentation updates** from Junio C Hamano aim to reduce contributor confusion, and **test modernizations** continue at a steady clip.

---

### Notable threads

#### **`git log --graph` cascading indentation: usability redesign debate**
**Headline:** *Mirko Faina’s bounded zig-zag counter-proposal shifts the design discussion from “whether to indent” to “how to indent minimally.”*

Pablo Sabater’s **v9 series** implementing cascading indentation for visual roots in `git log --graph` is **technically complete**, with all known edge cases (including `--author`/`-grep` filters) resolved. The series now faces a **late-stage usability debate** sparked by Mirko Faina, who argues that always-on indentation clutters output for simple cases (e.g., sequential one-patch series). Mirko’s **bounded zig-zag counter-proposal**—capping indentation at two levels before resetting to the left margin—offers a middle ground: it preserves clarity for excluded parents while avoiding excessive indentation in trivial histories. The proposal is **opt-in** (triggered by `--oneline` or `--format=reference`), addressing Mirko’s original objection to always-on behavior.

Pablo has not yet responded to the zig-zag idea, but the bounded variant appears to satisfy both the technical goal (disambiguating excluded parents) and the usability goal (avoiding clutter). The maintainers must now decide whether to:
- Adopt the bounded zig-zag as the default or opt-in behavior.
- Stick with opt-out cascading indentation (Pablo’s v9 design).
- Switch to opt-in cascading indentation (Pablo’s earlier alternative).
- Postpone the decision and address usability in a follow-up.

The series remains **ready for `next`** pending final reviews and a ruling on the usability question. The lookahead buffer infrastructure (patches 2–3/4) is uncontroversial and could land independently if the visualization logic is deferred.

---

#### **`git branch --delete-merged`: final mechanical cleanup**
**Headline:** *Harald Nordgren’s v18 series awaits only a flags cleanup before merging.*

Harald Nordgren’s **`--delete-merged`** series, which enables safe automated local branch cleanup with **per-branch opt-out**, **stacked-branch protection (abort-and-clear)**, and **`--dry-run` preview**, is **implementation-complete**. All substantive review feedback has been addressed, including:
- **Test improvements** (clearer branch names, helper functions, redundant command removal).
- **`<branch>` argument support** for `--delete-merged`, reusing the `--forked` filter machinery.
- **Stacked-branch protection** (abort-and-clear) to handle chains of dependencies (e.g., `b1→b2→b3`).

The **only remaining action** is a mechanical fix to patches 2–4/7: replacing local boolean variables (`force`, `quiet`) with direct flag checks (`flags & DELETE_BRANCH_FORCE`) to avoid two sources of truth. Harald has agreed to this cleanup, which does not affect functionality. The series is **ready for merging** once the fix is applied.

---

#### **`paint_down_to_common()` optimization: ready for `seen`**
**Headline:** *Tian Yuchen and Kristofer Karlsson’s v6 series clears its last procedural hurdle.*

The **performance optimization series** for `paint_down_to_common()`—delivering **100-1000x speedups** for asymmetric merge-base queries—is now **procedurally unblocked**. Junio C Hamano confirmed the series applies cleanly to a **synthetic base** (stable tag merged with `kk/commit-reach-find-all-fix`), resolving the last merge conflict. The series is **fully reviewed**, **regression-free**, and **self-verifying** (deterministic step counts, exhaustive edge-case coverage). Key changes:
- **Early termination** when one side’s commit queue is exhausted.
- **Trace2 instrumentation** to measure performance improvements.
- **Removal of the commit-date fallback** (introduced in 2018), simplifying termination conditions.

The series is **ready for queuing in `seen`** once Tian resubmits v6 with an updated cover letter clarifying the base. No code changes are needed.

---

#### **`USE_NSEC` deprecation: runtime config replaces build-time knob**
**Headline:** *Junio withdraws the original patch; consensus emerges on `core.useNsec` and deprecating `USE_NSEC`.*

The discussion around D. Ben Knoble’s patch to add a `nanosec` Meson option (mirroring Autotools’ `USE_NSEC`) has **evolved into a broader rethink** of how Git handles sub-second timestamps. The consensus now favors:
1. **Always compiling nanosecond support** into Git when the platform provides the necessary APIs (e.g., modern Linux).
2. **Exposing the feature via a runtime config knob** (`core.useNsec`), making it more accessible than the current build-time knob.
3. **Retaining the current build-time default** (`USE_NSEC` disabled) for backward compatibility.
4. **Deprecating and eventually removing the `USE_NSEC` build knob** (Meson and Makefile) once `core.useNsec` is implemented.

Jeff King’s testing confirmed that nanosecond timestamps are preserved correctly on modern Linux filesystems (ext4, XFS, vfat, CIFS, NTFS, FUSE), obsoleting historical "racy Git" concerns. The runtime config approach addresses interoperability risks (e.g., mixing `USE_NSEC` and non-`USE_NSEC` implementations) by allowing users to opt in or out as needed. The next step is a new patch series implementing `core.useNsec` and deprecating the build-time knob.

---

### In brief

- **`repo_config_values` migration (v9):** Tian Yuchen’s series to migrate nine global config variables into `struct repo_config_values` is **under review**, with all memory leaks fixed. Pablo Sabater’s review raised **design concerns** about submodule handling in `repo_config_values_clear()`, but Tian has agreed to align the signatures of `_clear()` and `_init()` (both will take `struct repository *`). The series is **CI-clean** and ready for v10.
- **`git-subtree` safeguard:** Ian Jackson’s patch to prevent mixing the shell-script implementation with a forthcoming Rust rewrite is **awaiting v2** with test style improvements (using `test_commit` and orphan branches). The Rust rewrite’s stance on manual `-X subtree` merges was clarified: it will neither support nor actively prevent them, but will document the risks of mixing them with the new implementation.
- **Commit-graph regression fix:** Kristofer Karlsson’s **post-merge discussion** about splitting failing tests and fixes into separate commits continues. Junio C Hamano and Taylor Blau debated the trade-off between author convenience and reviewer clarity, with no consensus on formalizing guidance in `Documentation/SubmittingPatches`.
- **ODB abstraction (v2):** Patrick Steinhardt’s series to add object filters to `odb_for_each_object()` is **under review**, with Taylor Blau and Jeff King identifying **performance inefficiencies** and **design questions** about traversal API consistency. Jeff’s follow-up cleanup patch (removing the unused `int` return from `add_object_entry()`) is ready to proceed.
- **`git add -e` refactoring (v4):** Gatla Vishweshwar Reddy’s patch to replace the subprocess call to `git apply` with the internal `apply_all_patches()` API is **ready for merging**, with all feedback addressed. The v4 update ensures correct subdirectory operation by passing `NULL` as the prefix to `init_apply_state()`.
- **Double-free fix:** Patrick Steinhardt’s bugfix for a double-free in the object-file subsystem’s stream handling is **ready to merge**, with Jeff King confirming the fix and test. The security implications were clarified: the bug is outside Git’s threat model, as affected code paths don’t operate on the transport layer.
- **`git log` date default:** The discussion about why Git defaults to showing the *author date* rather than the *committer date* continues, with Jeff King and Junio C Hamano explaining the rationale (author date reflects when the patch was *sent* to the mailing list, not when it was written). No consensus on changing the default or adding a config option (e.g., `log.defaultDate=committer`).
- **`fetch.submoduleErrors` test fixup:** Ramsay Jones’s mechanical fix to replace `grep` with `test_grep` in `t/t5526-fetch-submodules.sh` is **ready to merge**, addressing a regression in the `pz/fetch-submodule-errors-config` series.
- **Documentation updates:** Junio C Hamano’s six-patch series to clarify contributor guidance is **under review**. Key additions:
  - Commit message structure: **Observation**, **Solution**, **Command**.
  - Handling unresponsive patches: send a polite "ping" after 1–2 weeks.
  - `Reviewed-by:`/`Acked-by:` trailer carry-over rules.
  - `seen` branch is not an acceptance signal; "Will merge to `next`" is the actual signal.
  - "What's cooking" report style: past/present perfect tense in passive voice.

---

### On the radar
- **`git log --graph` usability:** The bounded zig-zag counter-proposal needs a maintainer ruling. Pablo Sabater’s response will determine whether the series lands as-is or requires a redesign.
- **`USE_NSEC` deprecation:** The runtime config approach (`core.useNsec`) needs a patch series to implement it and deprecate the build-time knob.
- **ODB abstraction:** Taylor Blau’s feedback on traversal API consistency may require coordination with other parts of the codebase. Jeff King’s cleanup patch is ready to proceed.