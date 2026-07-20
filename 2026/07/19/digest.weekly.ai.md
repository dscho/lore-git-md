# Git Mailing List Digest — 2026/07/13 to 2026/07/19

## The period in brief

The week of **2026/07/13 to 2026/07/19** was **exceptionally active**, with **over 700 emails across 150+ threads** and **major architectural efforts reaching critical milestones**. Traffic was **heavy and eventful**, with **three long-running series landing in `next`**, a **contentious design debate** in `git replay --linearize`, and **multiple blocking correctness issues** surfacing late in integration cycles. The two developments a reader absolutely should not miss:

- **`git branch --delete-merged` (Harald Nordgren) hit a blocking order-dependency bug** just as it was poised for `next`, threatening its core safety guarantees.
- **`git replay --linearize` (Toon Claes) remains stalled** after Junio C Hamano and Elijah Newren clashed over multi-branch behavior, with no resolution in sight.

The week also saw **ODB abstraction work graduate to `next`**, **`git history squash` land as a foundational rebase backend**, and **`git cat-file --batch-command` remote-object-info complete its GSoC journey** with robust security and protocol design.

---

## Key developments

### `git branch --delete-merged` blocked by order-dependency bug

**Author**: Harald Nordgren
**Status**: **Implementation-complete but blocked** by a **blocking correctness issue** in the stacked-branch protection algorithm.

The **19th iteration** of this long-running series introduced a **safety mechanism** to prevent deleting branches that are used as upstreams for unmerged branches, while clearing stale upstream configs for kept branches. However, Junio C Hamano demonstrated that the algorithm’s behavior depends on the **alphabetical order of branch names** rather than their logical dependency chain, violating the principle that identical relationships should produce identical results.

**Key participants**: Junio C Hamano, Phillip Wood, Harald Nordgren.
**Stage**: The series is **technically complete** and **functionally sound** for most use cases, but the **order-dependency bug** threatens its core safety guarantees. Phillip Wood proposed a **structural refactor** (collecting spared branches in a temporary `strset` during traversal, then removing them in a single batch operation) to eliminate the sequential, order-dependent behavior. Junio remains skeptical and has not yet signed off on the fix.

**Why it matters**: This series is a **major usability improvement** for Git’s branch management, enabling safe, automated cleanup of merged local branches with comprehensive filtering and safety controls. The stacked-branch protection is a **critical safety feature** for users with complex dependency chains, and the order-dependency bug undermines its reliability. The resolution of this issue will determine whether the series graduates to `next` or remains stalled.

---

### `git replay --linearize` design debate over multi-branch behavior

**Author**: Toon Claes
**Status**: **Technically complete but stalled** due to a **fundamental design dispute** over the `--linearize` option’s behavior when multiple positive refs are provided.

The **v7 series** introduces `--linearize`, which flattens all specified branches into a single linear history, concatenating them in an order determined by the revision walk (not the user’s command-line order). Elijah Newren argues this **violates the principle of least astonishment**, creates a **recoverability asymmetry**, and is an **emergent, unintended behavior** that neither feature implies. Junio C Hamano defends the current behavior as **intentional and reasonable**, framing the concatenation as a **feature** (users can avoid it by invoking `git replay` multiple times) and suggesting **syntax alternatives** to reduce ambiguity.

**Key participants**: Junio C Hamano, Elijah Newren, Toon Claes.
**Stage**: The series is **technically complete** and **queued in `next`**, but the **design debate** remains unresolved. Toon Claes has not yet responded to the competing visions, and the series is unlikely to graduate to `master` without a resolution. The outcome will set a precedent for how future commands handle **multi-branch operations** and whether **emergent behaviors** are acceptable if they align with user intent.

**Why it matters**: `--linearize` is a **simpler alternative** to Johannes Schindelin’s earlier merge-replay implementation, offering predictable, all-or-nothing flattening. The debate highlights a **tension between flexibility and predictability** in Git’s design, with implications for the project’s command-line UI philosophy.

---

### ODB abstraction work graduates to `next`

**Author**: Patrick Steinhardt
**Status**: **Two major series (`ps/odb-pluggable-housekeeping` and `ps/odb-for-each-object-filter`) are now queued in `next`**, marking significant progress toward pluggable backends.

#### `ps/odb-pluggable-housekeeping` (12 patches)
Refactors Git’s object housekeeping (repacking, geometric repacking, auto-gc) to be backend-agnostic, enabling future ODB backends (e.g., reftable) to implement their own optimization strategies. Key changes:
- Moves ODB optimization logic from `builtin/gc.c` to `odb/source-files.c`.
- Introduces `struct odb_optimize_options` to encapsulate settings.
- Unifies incremental and geometric repacking under a single `odb_optimize()` entry point.
- Expands the "pre-auto-gc" hook to intercept all auto-maintenance tasks, not just `gc`.

#### `ps/odb-for-each-object-filter` (9 patches)
Adds object filters to `odb_for_each_object()`, enabling `git cat-file --batch-all-objects` to work with pluggable backends. Key changes:
- Extends `odb_for_each_object()` with a `filter` parameter.
- Teaches the "packed" backend to skip excluded objects via bitmaps.
- Removes direct ODB backend access from `git cat-file`, replacing it with the generic interface.

**Key participants**: Junio C Hamano, Patrick Steinhardt.
**Stage**: Both series are **technically complete**, **approved for `next`**, and **queued for integration testing**. No substantive objections remain.

**Why it matters**: These series are **prerequisites for pluggable ODB backends**, allowing Git to support alternative storage systems (e.g., cloud-based object stores) without backend-specific hacks. The refactoring is **performance-neutral** for existing workflows but enables future optimizations and architectural flexibility.

---

### `git history squash` lands as a rebase backend

**Author**: Harald Nordgren
**Status**: The **9th and final iteration** of this series is **technically complete** and **queued for `next`**, with all prior feedback addressed.

The series implements `git history squash`, a new command that efficiently collapses a linear or merge-commit-containing range into its oldest commit while preserving descendant history. Key features:
- **Autosquash alignment**: The `--reedit-message` template now **exactly matches `git rebase -i --autosquash`**, grouping `fixup!`/`squash!`/`amend!` commits under their targets.
- **Rev-list option validation**: Warns if user-provided options (e.g., `--reverse`) would alter the walk order, preventing silently incorrect results.
- **Ref-handling**: Rejects operations with interior refs by default, with advice to use `--update-refs=head`.

**Key participants**: Junio C Hamano, Matt Hunter, Phillip Wood.
**Stage**: The series is **functionally complete** and **ready for integration**. Junio C Hamano **fully endorsed** the vision of using `git history squash` and `git replay` as a **faster, non-interactive backend for `git rebase --autosquash`**, calling it "an ideal future." This series is a **foundational piece** of that effort, eliminating the repeated conflict resolution of the current rebase implementation.

**Why it matters**: This command is a **major architectural shift** toward modular, conflict-free history editing. The eventual goal is to replace `git rebase -i`’s internals entirely, and this series is a **critical enabler** of that vision.

---

### `git cat-file --batch-command` remote-object-info graduates to `next`

**Author**: Pablo Sabater
**Status**: The **20th and final iteration** of this GSoC series is **technically complete** and **queued for `next`**, completing a project that enables secure, metadata-only queries of remote objects (e.g., size) without downloading full objects.

Key features:
- **Dynamic allow-list** for metadata fields (e.g., `objectsize`), ensuring clients only request what the server supports.
- **Strict input validation**: 10,000-object batch limit, 8KB URL length cap, and bounds-checked parsing.
- **699 lines of new tests** covering edge cases (missing objects, unsupported placeholders, transport compatibility).
- **Protocol v2 extension**: The `object-info` capability advertises supported attributes when `transfer.advertiseobjectinfo` is enabled.

**Key participants**: Junio C Hamano, Jeff King, SZEDER Gábor.
**Stage**: The series is **technically complete**, with all build issues resolved and no substantive objections remaining. It is now **queued for `next`** pending minor cosmetic nits (translation context, alphabetical ordering).

**Why it matters**: This is the **plumbing for a long-awaited feature**—efficient object metadata queries without full downloads—critical for partial clones and write-through proxies. The dynamic capability negotiation ensures forward compatibility as the protocol expands.

---

### Sequencer regression: dropped-commit notes still copied in interactive rebase

**Author**: Phillip Wood
**Status**: The **v3 series fixing how `git rebase` handles notes from dropped commits** is **queued for `next`**, but a **blocking regression** was reported by Uwe Kleine-König: notes from dropped commits are still copied in interactive rebase (`-i`), though the fix works for non-interactive rebase.

**Key participants**: Junio C Hamano, Phillip Wood, Uwe Kleine-König.
**Stage**: The series is **technically complete** and **approved for `next`**, but the **regression** undermines its core behavioral fix. Phillip Wood clarified that the series only handles commits dropped by `--empty=drop` (non-interactive default), not **user-initiated drops** (e.g., `git rebase --skip` or `--continue` without committing). The interactive backend’s `--empty=ask` quirk (drops on `--continue`, keeps on `--skip`) further complicates the fix.

**Why it matters**: This **eliminates a long-standing source of corruption** in rebase operations, particularly for users of `git notes` or `post-rewrite` hooks. The regression is a **high-priority follow-up**, and Phillip is designing a fix to address the gap, which stems from the interactive backend’s behavior.

---

## In brief

### Features and enhancements
- **`git fetch --submodule-errors`** (Paulius Zaleckas) – **Ready for `next`**. Adds `fetch.submoduleErrors` config and `--submodule-errors` CLI to control whether submodule fetch failures abort the parent fetch.
- **`git show-branch` slab conversion** (Gatla Vishweshwar Reddy) – **Ready for `next`**. Replaces `object.flags` with a commit-slab (`commit_rev_flags`) using runtime-computed stride, removing the 27-branch limit.
- **`git bisect --auto-reset`** (Harald Nordgren) – **v2 posted**. Implements `--auto-reset=<where>` (values `original` or `found`) to control where the working tree resets after bisect.
- **`git history` signing** (Souma) – **v2 posted**. Teaches `git history` to sign rewritten commits, addressing Patrick Steinhardt’s v1 feedback.
- **`git repo info` path keys** (K Jayatheerth) – **v2 posted**. Adds 18 new path-related keys (e.g., `path.toplevel.absolute`, `path.git-prefix`).
- **`git last-modified` Bloom filter optimization** (Toon Claes) – **v1 posted**. Reuses Bloom filters to optimize `git last-modified`, yielding **4× speedups** (64 ms → 16 ms) on the ziglang/zig repository.

### Bugfixes
- **Packfile URI races** (Ted Nyman) – **Design pivot underway**. Fixes race conditions in Git’s HTTP transport when multiple processes fetch the same packfile URI, with a focus on preserving resumable downloads.
- **Reftable tombstone performance** (Kristofer Karlsson) – **Merged to `master`**. Fixes a quadratic performance regression in the reftable backend, improving performance from ~14s to ~0.2s in 8000-ref scenarios.
- **`git rebase --autosquash` empty commits** (Farid Zakaria) – **Ready for integration**. Makes `--autosquash` respect `--empty` for emptied commits.
- **Cygwin IPv6 SCP URLs** (Ramsay Jones) – **Queued in `seen`**. Fixes a 5-year-old regression in IPv6-style SCP URL handling.
- **Trace2 timestamp crash** (Derrick Stolee) – **Under review**. Fixes a crash in trace2 telemetry when timestamp formatting fails.
- **`git rev-list --no-walk` path filtering regression** (Peter Colberg) – **Root cause identified**. Kristofer Karlsson proposed a fix to restore `process_parents()` in the `REV_WALK_NO_WALK` case.

### Refactoring and cleanup
- **`the_repository` removal** (Tian Yuchen, René Scharfe) – **Multiple series queued for `next`**. Migrates global config variables into `struct repo_config_values` and introduces repository-aware tempfile/lockfile APIs.
- **Test modernization** (Shlok Kulshreshtha) – **Queued for `next`**. Modernizes `t/t1100-commit-tree-options.sh` to use modern Git test style conventions.
- **`git fast-export` help text** (Christian Couder) – **Queued in `seen`**. Aligns the `git fast-export -h` usage string with the man page and Git conventions.
- **`strbuf_getwholeline()` refactoring** (René Scharfe) – **Queued in `next`**. Removes a redundant `strbuf_reset()` call in the `HAVE_GETDELIM` code path.

### Documentation
- **Contributor guidance** (Junio C Hamano) – **Under review**. Clarifies contributor guidance in `Documentation/MyFirstContribution.adoc` and `Documentation/SubmittingPatches`.
- **Documentation synopsis conversion** (Jean-Noël Avila) – **Under review**. Converts `git imap-send`, `git format-patch`, `git send-email`, and `git request-pull` to the new AsciiDoc synopsis style.

---

## Looking ahead

The next week is likely to be dominated by **resolutions to the blocking issues** that surfaced this week:

- **`git branch --delete-merged`**: Harald Nordgren must address the **order-dependency bug** in the stacked-branch protection algorithm. Phillip Wood’s proposed refactor offers a concrete path forward, but Junio’s satisfaction is the final hurdle.
- **`git replay --linearize`**: Toon Claes must respond to the **design debate** between Junio and Elijah. The resolution will determine whether the series graduates to `master` or remains stalled.
- **Sequencer regression**: Phillip Wood is designing a fix for the **user-initiated drops in interactive rebase**, which will address the regression in the recently approved series.
- **`git last-modified` Bloom filter optimization**: Toon Claes must address the **correctness bugs** and **test gaps** identified by Taylor Blau and Jeff King.

Additionally, **ODB abstraction work** will continue to progress, with Patrick Steinhardt’s loose-object backend refactoring now unblocked and ready for review. The **Rustification effort** (Ezekiel Newren) and **reftable backend** (Patrick Steinhardt, Karthik Nayak) remain ongoing, with no major milestones expected in the immediate future.

The **`no-ref-delta` capability** design debate (Jeff King, Taylor Blau) is likely to continue, with implications for the write-through proxy’s performance and compatibility. The **linked files proposal** (sporteka) could spark broader discussion about whether such guarantees belong in core Git or remain workflow-specific.