Here is the digest for **2026-07-13**:

---

### The day in brief
A **heavy but focused** day on the Git mailing list, with **130 emails across 34 threads**. The standout developments: **Pablo Sabater’s `git log --graph` visual root indentation series (v11) cleared its final review hurdles** and is poised for `next`; **Phillip Wood’s sequencer-dropped-commits overhaul graduated to `master`**; and **Elijah Newren raised a substantive objection to `git replay --linearize`**, holding it in `next` pending resolution. The day also saw **new bugfix series for packfile URI races**, **test modernizations**, and **clarifications on stalled topics**.

---

### Notable threads

#### **`git log --graph` visual root indentation (v11) – Pablo Sabater**
The **eleventh iteration** of Pablo Sabater’s feature series—adding indentation to `git log --graph` to clarify ancestry relationships for commits with excluded parents—**cleared its final review hurdles**. Junio Hamano and Mirko Faina approved the mechanical bit-packing tweaks in the opt-out mechanism (`--[no-]graph-indent` and `log.graphIndent`), and the series is now **technically complete and ready for `next`**. The only deferred work is reconciliation with Kristofer Haugsbakk’s `kk/prio-queue-get-put-fusion` topic, but this is not a blocker. The series introduces a **lookahead buffer** to safely determine visual root status, a **four-level wrapping** mechanism to prevent excessive indentation, and **per-branch opt-out** via config or CLI. **Key takeaway**: This long-running effort (11 revisions) is now on the cusp of integration, offering a cleaner graph layout for complex histories.

#### **Sequencer-dropped-commits overhaul (v2) – Phillip Wood**
Phillip Wood’s **10-patch systemic overhaul** of the sequencer’s handling of dropped commits **graduated to `master`**. The series fixes a long-standing bug where `git rebase` incorrectly copied notes from dropped commits to the current HEAD, and expands to handle edge cases like **external merge strategy failures** and **command execution failures**. Junio Hamano’s final review confirmed the series is **technically sound**, and the only remaining action is squashing queued typofixes. **Key takeaway**: This is a **high-quality, tested-level contribution** that directly tackles technical debt in the rebase machinery, with no user-visible behavior changes.

#### **`git replay --linearize` objection – Elijah Newren**
Elijah Newren **raised a substantive objection** to Toon Claes’s `git replay --linearize` feature, which was recently merged to `next` and marked for `master`. The objection centers on a **principle-of-least-astonishment violation**: when multiple positive refs are provided (e.g., `git replay --linearize --onto main topic1 topic2`), the current implementation flattens all commits into a single linear history, concatenating branches in an order determined by the revision walk. Newren argues this is inconsistent with other `git replay` modes (`--advance` and `--revert`), which reject multiple positive refs to avoid ambiguity. Junio Hamano **pushed back**, arguing that unpredictable order is inherent to linearization and that the current behavior is acceptable for most use cases. **Key takeaway**: The topic is **held in `next` pending resolution**, with the ball in Toon Claes’s court to either document the behavior, restrict multiple refs, or propose a middle ground.

#### **Packfile URI download races – Ted Nyman**
Ted Nyman posted a **two-patch bugfix series** addressing race conditions in Git’s HTTP transport when multiple processes concurrently fetch the same packfile URI. The patches:
1. Use **unique temporary files** for each packfile URI download, preventing concurrent corruption.
2. Make `fetch-pack` accept both `"pack<TAB><hash>"` and `"keep<TAB><hash>"` success messages from `index-pack`, avoiding spurious failures when `.keep` files already exist.
The series includes **regression tests** and is narrowly scoped to the packfile URI feature. **Key takeaway**: This is a **well-motivated, uncontroversial fix** for a real-world edge case, with no objections raised yet.

#### **`git branch --delete-merged` (v18) – Harald Nordgren**
Harald Nordgren’s **eighteenth iteration** of the `git branch --delete-merged` series **addressed all remaining review feedback**, including mechanical fixes for the flags-based API and rejecting the nonsensical `--dry-run --quiet` combination. The series is now **implementation-complete and ready for merging**, pending Junio Hamano’s final review. **Key takeaway**: This long-running effort (18 revisions) introduces **safe automated local branch cleanup** with comprehensive filtering and safety controls, including **per-branch opt-out** and **stacked-branch protection**.

#### **ODB abstraction efforts – Patrick Steinhardt**
Patrick Steinhardt’s **ODB abstraction efforts** saw progress on multiple fronts:
- **`ps/odb-pluggable-housekeeping` (v2)**: The 12-patch series refactoring `git gc`/`maintenance` ODB housekeeping to be backend-agnostic **addressed Junio’s feedback** and is now **ready for integration**. The series moves files-backend logic to `odb/files-backend.c` and introduces `odb_optimize_required()`.
- **`ps/odb-for-each-object-filter` (v3)**: The 9-patch series adding object filters to `odb_for_each_object()` **incorporated Jeff King’s cleanup patch** and is now **technically complete**. The series enables `git cat-file --batch-all-objects` to use the generic interface, eliminating direct ODB backend access.
**Key takeaway**: These series are **prerequisites for pluggable ODB backends**, with no substantive objections remaining.

---

### In brief

- **`includeIf worktree:<pattern>` (v8) – Chen Linxuan**: Patrick Steinhardt **approved the series**, clearing it for `next`. The feature enables conditional config inclusion based on the worktree’s path, with a documented limitation around symlinks.
- **`git checkout --track=fetch` (v4) – Harald Nordgren**: The series **remains stalled** pending further review, despite the author’s claim that all feedback has been addressed.
- **`git fast-export` help text alignment – Christian Couder**: Junio Hamano **queued the patch** to align `git fast-export -h` with the man-page SYNOPSIS and Git conventions.
- **Test modernizations – Shlok Kulshreshtha**: A new contributor’s **two-patch series** modernizing `t/t1100-commit-tree-options.sh` received **surface-level review** from Junio, who requested a minor commit message tweak.
- **`git rebase --autosquash` empty commits (v3) – Farid Zakaria**: The series **awaits graduation of its dependency** (`pw/rebase-drop-notes-with-commit`) to `next` before rebasing.
- **Cygwin IPv6 SCP URL regression – Ramsay Jones**: The patch **remains under discussion**, with Ramsay planning to mark failing tests as Cygwin-specific skips.
- **Documentation contributor guidance (v1) – Junio C Hamano**: The six-patch series clarifying contributor guidance **addressed feedback** from Weijie Yuan and Michael Montalbo, with a reroll pending minor editorial tweaks.

---

### On the radar

- **`git replay --linearize`**: Held in `next` pending resolution of Elijah Newren’s objection.
- **`mm/diff-process-hunks`**: Michael Montalbo **announced a reroll** of the stalled topic within the week.
- **`tb/repack-geometric-cruft`**: The 11-patch series combining geometric and cruft repacking **needs a reroll** for MIDX retention logic.
- **`ps/cat-file-remote-object-info`**: The 13-patch series adding `remote-object-info` to `git cat-file --batch-command` **may face protocol feature-creep concerns**.

---

### Editorial note
Today’s traffic was **heavy but focused**, with several long-running series reaching key milestones. The **`git log --graph` indentation series** and **sequencer-dropped-commits overhaul** stand out as **high-quality, tested-level contributions** that directly address user pain points. The **`git replay --linearize` objection** is a reminder that even well-reviewed features can face late-stage design challenges, and the **packfile URI race fixes** highlight the importance of edge-case testing in distributed workflows. The day’s discussions also underscored the **ongoing tension between flexibility and predictability** in Git’s UI design, a theme likely to recur as the project evolves.