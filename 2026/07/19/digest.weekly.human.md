# Git Mailing List Digest: 2026/07/13 -- 2026/07/19

## The period in brief

The week of **2026/07/13--2026/07/19** was **exceptionally active** (620 emails across 152 threads), with **three major features reaching integration readiness**, **two long-running architectural efforts landing in `next`**, and **multiple design debates** surfacing around usability, safety, and protocol design. The standout developments: **`git cat-file --batch-command remote-object-info` graduated to `next`**, **`git history squash` landed as a foundational piece of Git’s future rebase architecture**, and **ODB abstraction work saw two series approved for `next`**. A **contentious debate over `git replay --linearize`’s multi-branch behavior** remains unresolved, while **correctness bugs in `git branch --delete-merged` and the sequencer’s dropped-commit handling** blocked their respective series.

---

## Key developments

### `git cat-file --batch-command` remote-object-info graduates to `next`
**Author**: Pablo Sabater
**Status**: **Technically complete and queued for `next`** after 20 iterations.

The series implements a **security-hardened protocol v2 extension** that lets clients query object metadata (size, with type support designed in) from remotes without downloading full objects. Key features:
- **Dynamic allow-list** for metadata fields (e.g., `objectsize`), ensuring clients only request what the server supports.
- **Strict input validation**: 10,000-object batch limit, 8KB URL length cap, and bounds-checked parsing.
- **747 lines of new tests** covering edge cases (missing objects, unsupported placeholders, transport compatibility).

**Why it matters**: This is a **critical enabler for pluggable ODB backends**, allowing tools like `git fsck` or `git gc` to work efficiently with partial clones or alternative storage systems. The protocol’s design (e.g., omitting unsupported placeholders like `%(objectcolor)`) aligns with Git’s established patterns, and the security hardening makes it safe for production use.

---

### `git history squash` lands as a rebase backend
**Author**: Harald Nordgren
**Status**: **Technically complete and queued for `next`** (v9).

The series implements `git history squash`, a new command that efficiently collapses a linear or merge-commit-containing range into its oldest commit while preserving descendant history. Key features:
- **Autosquash alignment**: The `--reedit-message` template now **exactly matches `git rebase -i --autosquash`**, grouping `fixup!`/`squash!`/`amend!` commits under their targets.
- **Rev-list option validation**: Warns if user-provided options (e.g., `--reverse`) would alter the walk order, preventing silently incorrect results.
- **Ref-handling**: Rejects operations with interior refs by default, with advice to use `--update-refs=head`.

**Why it matters**: Junio C Hamano **fully endorsed** the vision of using `git history squash` and `git replay` as a **faster, non-interactive backend for `git rebase --autosquash`**, calling it "an ideal future." This series is a **foundational piece** of that effort, eliminating the repeated conflict resolution of the current rebase implementation.

---

### ODB abstraction: two series approved for `next`
**Author**: Patrick Steinhardt
**Status**: Both series are **technically complete and queued for `next`**.

#### `ps/odb-pluggable-housekeeping` (12 patches)
Refactors Git’s object housekeeping (repacking, geometric repacking, auto-gc) to be backend-agnostic, enabling future ODB backends (e.g., reftable) to implement their own optimization strategies. Key changes:
- Moves ODB optimization logic from `builtin/gc.c` to `odb/source-files.c`.
- Introduces `struct odb_optimize_options` to encapsulate settings.
- Unifies incremental and geometric repacking under a single `odb_optimize()` entry point.

#### `ps/odb-for-each-object-filter` (9 patches)
Adds object filters to `odb_for_each_object()`, enabling `git cat-file --batch-all-objects` to work with pluggable backends. Key changes:
- Extends `odb_for_each_object()` with a `filter` parameter.
- Teaches the "packed" backend to skip excluded objects via bitmaps.
- Removes direct ODB backend access from `git cat-file`, replacing it with the generic interface.

**Why it matters**: These series are **prerequisites for pluggable ODB backends** and represent **major architectural progress**. The refactoring is **performance-neutral** for existing workflows but enables future optimizations.

---

### `git replay --linearize`’s multi-branch ambiguity remains unresolved
**Author**: Toon Claes
**Status**: **Technically complete (v7) but blocked by a design debate**.

The series introduces `--linearize`, a simpler alternative to Johannes Schindelin’s earlier merge-replay implementation, offering predictable, all-or-nothing flattening. However, a **philosophical debate** between **flexibility (Junio C Hamano)** and **strictness (Elijah Newren)** has stalled progress. The current implementation flattens all specified branches into a single linear history, concatenating them in an order determined by the revision walk (not the user’s command-line order).

### Key positions

- **Junio**: The current behavior is **intentional and reasonable**. Users can avoid concatenation by invoking `git replay` multiple times.
- **Elijah**: The current behavior **violates the principle of least astonishment**, creates a **recoverability asymmetry**, and is an **emergent, unintended behavior**. He proposes a **minimal fix**: restrict `--linearize` to a single positive ref.

**Why it matters**: This debate **highlights a tension** between flexibility and predictability in Git’s design. The outcome will set a precedent for how future commands handle **multi-branch operations**.

---

### `git branch --delete-merged` blocked by order-dependency bug
**Author**: Harald Nordgren
**Status**: **Implementation-complete (v19) but blocked by a correctness issue**.

The series introduces `--delete-merged`, a new subcommand that safely deletes local branches whose work has already landed on their upstream, while preserving branches that are still needed (e.g., checked-out, used as upstreams for unmerged branches, or opted out via config). Key features:
- **Repeatable upstream selector**: The `--delete-merged` option can now be specified multiple times to widen the upstream match.
- **Refined stacked-branch protection**: If a branch is used as an upstream for an unmerged branch, it is spared, and its own stale upstream config is cleared.
- **Per-branch opt-out**: Users can exempt specific branches from automated cleanup via `branch.<name>.deleteMerged=false`.
- **`--dry-run` preview**: Prints which branches would be deleted without modifying refs or config.

**The blocker**: Junio C Hamano demonstrated that the algorithm’s behavior depends on the **alphabetical order** of branch names rather than their logical dependency chain. In two equivalent dependency graphs (`a_tip→b_mid→c_lower` vs. `tip→mid→lower`), the outcome (which branches are spared or have their upstream cleared) differed solely due to naming.

**Why it matters**: This **undermines the series’ core safety guarantee**. The fix—likely a **multi-pass algorithm**—must be implemented before the series can graduate to `next`.

---

### Sequencer regression: dropped-commit notes still copied in interactive rebase
**Author**: Phillip Wood
**Status**: **Approved for `next` (v3) but with a known regression**.

The series fixes how the rebase machinery handles dropped commits, ensuring notes and post-rewrite hooks no longer receive spurious data. Key changes:
- **`enum pick_result`** to distinguish dropped commits from successful picks.
- **Error handling** for external merge strategies and command execution failures.
- **Test coverage** for edge cases (conflict resolution, multiple fixups, missing pick commands).

**The regression**: The fix does not cover **user-initiated drops** in interactive rebase (e.g., `git rebase --skip` or `--continue` without committing). Phillip is designing a follow-up to address this gap, which stems from the interactive backend’s `--empty=ask` behavior (drops on `--continue`, keeps on `--skip`).

**Why it matters**: This **eliminates a long-standing source of corruption** in rebase operations, particularly for users of `git notes` or `post-rewrite` hooks.

---

## In brief

**`git log --graph` visual root indentation** (Pablo Sabater) -- **Ready for `next`** (v12). Introduces indentation for "visual roots" (commits with excluded parents) in `git log --graph` to clarify ancestry relationships. Features a lookahead buffer, four-level wrap, and opt-out mechanism.

**`the_repository` removal efforts** -- Two series advanced:
- **Tian Yuchen’s v12** (migrating global config variables into `struct repo_config_values`) is **ready for `next`**.
- **René Scharfe’s 5-patch series** (introducing repository-aware tempfile/lockfile APIs) is **queued in `seen`**.

**`git last-modified` Bloom filter optimization** (Toon Claes) -- **v1 posted, bugs identified**. Aims to optimize `git last-modified` using Bloom filters but has **correctness bugs** (e.g., `--show-trees` handling) and **test gaps**.

**`git stash rename`** (Emin Özata) -- **Contested**. Junio C Hamano dismissed its justification as insufficient, while Brian m. Carlson provided a concrete use case. The **naming** (`rename` vs `reword`) and **behavior** remain unresolved.

**`git bisect --auto-reset`** (Harald Nordgren) -- **Redesign in progress**. Junio prefers leaving the working tree on the first bad commit (the culprit) rather than resetting to the original pre-bisect commit.

**`git repack --drop-filtered`** (Siddharth Shrimali) -- **Stalled**. Junio identified a **logical flaw** in the `--drop-filtered` vs `--write-bitmap-index` validation. The safety mechanism is ineffective.

**`diff.<driver>.process`** (Michael Montalbo) -- **Technically complete (v5)**. Enables external tools to inject diff hunks into Git’s pipeline. Awaiting Junio’s sign-off.

**`git show-branch` slab conversion** (Gatla Vishweshwar Reddy) -- **Cultural pushback**. Junio escalated a critique about AI-generated contributions, delaying integration.

**`wt-status` performance** (Sahitya Chandra) -- **Landed**. Replaces O(n²) `string_list_insert()` calls with O(n log n) append-and-sort.

**`git history fixup` worktree bug** (Toon Claes) -- **New bug report**. Demonstrates that `git history fixup` corrupts the index in other worktrees.

---

## Looking ahead

- **`git replay --linearize`**: Toon Claes must respond to the design debate between Junio and Elijah. The series remains in `next` but is unlikely to graduate without resolution.
- **`git branch --delete-merged`**: Harald Nordgren must implement a **multi-pass algorithm** to eliminate the order-dependency bug. The series is otherwise ready for `next`.
- **`git repack --drop-filtered`**: Siddharth Shrimali must address Junio’s feedback about the `--drop-filtered` vs `--write-bitmap-index` validation flaw.
- **`git last-modified` Bloom filter optimization**: Toon Claes must fix the **correctness bugs** and add **test coverage** for wildcard pathspecs.
- **ODB abstraction**: Patrick Steinhardt’s loose-object backend refactoring is **unblocked** and ready for substantive review.