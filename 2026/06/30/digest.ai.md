**The day in brief.**
June 30, 2026 was a **heavy but focused** day on the Git mailing list, with **94 emails across 23 threads**. The standout developments were **Phillip Wood’s 11-patch series fixing the rebase-dropped-commit bug** (a long-standing issue where notes were incorrectly copied from dropped commits) and **Patrick Steinhardt’s 13-patch refactoring of repository setup** (separating discovery from configuration). Both efforts are now **ready for final review and merging**, marking significant progress in Git’s core infrastructure. Other notable threads included **design debates on `git replay --linearize`’s CLI consistency**, **usability discussions around `git history squash`**, and **security hardening for the reftable backend**.

---

### Notable threads

#### **Rebase-dropped-commit bug finally fixed**
**Subject:** [PATCH 0/11] sequencer: avoid copying notes from dropped commits during rebase
**Author:** Phillip Wood

Phillip Wood delivered a **comprehensive 11-patch series** that overhauls how the sequencer handles dropped commits during rebase, fixing a long-standing bug where notes from dropped commits were incorrectly copied to the current HEAD. The series also addresses three additional rebase bugs: external merge strategy failures, command execution failures, and final fixup cleanup. The patches are **well-structured**, with preparatory refactorings followed by the core fix, and include **expanded test coverage** to catch regressions. Junio C Hamano has already queued a handful of typofix commits on top of the series, signaling readiness for merging. This is a **major milestone** for Git’s rebase machinery, resolving a correctness issue that has lingered since 2018.

---

#### **Repository setup refactoring nears completion**
**Subject:** [PATCH 0/13] setup: separate repository discovery from setup
**Author:** Patrick Steinhardt

Patrick Steinhardt’s **13-patch series** to separate repository discovery from setup is now complete and ready for review. The series introduces a new `struct repo_discovery` to hold discovery results (gitdir, worktree, prefix, and format), making the two phases distinct and self-contained. This refactoring is part of the broader effort to **consolidate repository configuration into `repo_init()`** and reduce scattered setup logic. The patches are **mechanical but foundational**, touching core files like `setup.c`, `repository.c`, and several built-ins. Junio C Hamano has already fixed a minor typo in one commit, and the series is **poised to land in `next`** once final reviews are complete. This is a key step toward the long-term goal of **eliminating global state** in Git’s setup code.

---

#### **`git replay --linearize` CLI debate continues**
**Subject:** [PATCH v5 0/3] replay: introduce --linearize option
**Authors:** Toon Claes, Johannes Schindelin, Patrick Steinhardt

The design discussion around `git replay --linearize`’s CLI consistency with `git rebase` remains unresolved. Johannes Schindelin **rejected Patrick Steinhardt’s proposal** to mirror `git rebase`’s `--rebase-merges=<mode>` syntax, arguing that `git rebase`’s interface is "full of usability warts" and that `--linearize` should remain a standalone flag. Patrick acknowledged the usability concerns but insisted that any divergence from `git rebase` must be **explicitly justified** in the commit message. Toon Claes, the author, has aligned with Schindelin’s technical direction (restoring the `replayed_base` parameter to fix a regression) but has not yet addressed the CLI debate. The thread is now **split between algorithmic correctness** (resolved) and **interface design** (still open), with the latter likely requiring a maintainer decision before the feature can graduate.

---

#### **`git history squash` usability feedback surfaces recoverability concerns**
**Subject:** [PATCH v6 0/4] history: add squash subcommand
**Authors:** Harald Nordgren, Phillip Wood, Matt Hunter

User feedback on the newly merged `git history squash` feature has **expanded the discussion beyond template formatting** to broader usability concerns. Matt Hunter and Phillip Wood raised **recoverability issues**: `git reset --hard` may not fully undo a `git history` operation if `--update-refs` moved other branches, and Git lacks a way to atomically undo multi-ref operations. Harald Nordgren proposed a **human-centered reflog redesign** to better track high-level workflows, while Phillip suggested looking to tools like Jujutsu and git-branchless for inspiration. The thread has **shifted from implementation details to systemic UX gaps**, though no concrete patches have emerged yet. The `--reedit-message` template debate was resolved in favor of Phillip Wood’s cleaner design, but the recoverability discussion remains open.

---

#### **ODB abstraction stack advances**
**Subject:** [PATCH v2 0/2] odb: generalize reprepare() callback for pluggable ODBs
**Author:** Patrick Steinhardt

Patrick Steinhardt’s **3-patch series** to generalize the `reprepare()` callback into a more flexible `prepare()` callback has received **final approval** from Toon Claes and Junio C Hamano. The series enables `git grep` to work with pluggable ODB backends by decoupling cache invalidation from ODB preparation. It depends on `ps/odb-source-packed` (already merged to `next`) and is now **ready to advance**. This is another step toward **backend-agnostic object storage**, a key architectural goal for Git’s future.

---

### In brief

**Prio-queue optimization merged** -- Junio C Hamano signed off on Kristofer Karlsson’s **4-patch series** optimizing Git’s priority queue implementation, which folds the `lazy_queue` pattern into core functionality and maintains 1.7-2.7% speedups on traversal-heavy operations. The series is now merged.

**Reftable hardening series complete** -- Patrick Steinhardt’s **12-patch series** hardening the reftable backend against corrupted files is now complete, with all fixes and fuzzing infrastructure in place. Junio acknowledged the test helper refactoring, and the series is **ready for final review**.

**`git refs` subcommands merged** -- Patrick Steinhardt’s **5-patch series** adding `delete`, `update`, `create`, and `rename` subcommands to `git refs` is now merged to `next`. The series consolidates reference manipulation under a unified interface, improving discoverability.

**Memory leak fixes** -- Jeff King posted a **2-patch series** fixing memory leaks in `git format-patch` and the test harness. The patches are **self-contained and low-risk**, addressing issues discovered opportunistically.

**Test modernization** -- Bryan B. Lima’s patch modernizing `t7412-submodule-absorbgitdirs.sh` to use descriptive test helpers (`test_path_is_file`, etc.) was **accepted and queued** by Junio C Hamano.

**`git blame -b` usability fix** -- René Scharfe posted a **patch** to fix a usability friction in `git blame -b`, where the output reserved an extra hex digit for an unused caret marker. The patch refactors the mark-handling logic to align the abbreviated commit hash length with the user’s `core.abbrev` setting.

**Config parser case-sensitivity bug fixed** -- Rishav Dewan posted a **patch** fixing a long-standing case-sensitivity mismatch in Git’s config file parser, where updates to old-style `[section.subsection]` headers failed if the subsection contained uppercase letters. The patch adds a `subsection_case_sensitive` flag to handle case-sensitive and case-insensitive comparisons appropriately.

---

### On the radar

**`USE_NSEC` default flip and runtime configuration** -- The discussion around flipping the default of `USE_NSEC` to `true` and converting it to a runtime-configurable setting remains unresolved. Jeff King’s testing confirmed that modern Linux filesystems preserve nanosecond timestamps correctly, but interoperability risks (e.g., with JGit) and the lack of a portable auto-detection mechanism complicate the decision. No patches have been posted yet, but the thread is **worth tracking** for its implications on Git’s timestamp handling.

**`git clone --only/--except` for sparse-checkout** -- Pushkar Singh’s RFC proposing `--only`/`--except` options for `git clone` to specify sparse-checkout paths received **cautious feedback** from Jeff King, who suggested an alternative design (providing a patterns file at clone time). The proposal remains at the **design stage**, with no implementation yet. If the feature gains traction, it could simplify monorepo workflows by eliminating a separate `git sparse-checkout set` step.