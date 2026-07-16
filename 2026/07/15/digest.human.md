# Here is the digest for July 15, 2026:

---

## The day in brief

A busy day on the Git mailing list—**153 emails across 27 threads**—with a strong focus on **integration-ready features**, **bugfixes**, and **refactoring**. The standout developments: **`git cat-file --batch-command remote-object-info` graduates to `next`**, **`git history squash` lands as a foundational piece of Git’s future rebase architecture**, and **`git replay --linearize` nears resolution after a design debate**. Several long-running series (ODB abstraction, `the_repository` removal, reftable hardening) also saw final polish before merging.

---

## Notable threads

### **`git cat-file --batch-command remote-object-info` graduates to `next`**
**Thread**: [GSoC v18] `git cat-file --batch-command` remote-object-info
**Author**: Pablo Sabater
**Status**: **Technically complete and queued for `next`** after 18 iterations.

The **final blocker**—a silent failure mode in `parse_cmd_remote_object_info()`—was resolved by replacing a `goto cleanup` with `die()`, ensuring loud failures. The series now implements a **security-hardened protocol v2 extension** that lets clients query object metadata (size, with type support designed in) from remotes without downloading full objects. Key features:
- **Dynamic allow-list** for metadata fields (e.g., `objectsize`), ensuring clients only request what the server supports.
- **Strict input validation**: 10,000-object batch limit, 8KB URL length cap, and bounds-checked parsing.
- **699 lines of new tests** covering edge cases (missing objects, unsupported placeholders, transport compatibility).

**Why it matters**: This is a **critical enabler for pluggable ODB backends**, allowing tools like `git fsck` or `git gc` to work efficiently with partial clones or alternative storage systems. The protocol’s design (e.g., omitting unsupported placeholders like `%(objectcolor)`) aligns with Git’s established patterns, and the security hardening makes it safe for production use.

**Next steps**: Junio’s "What’s cooking" report will likely confirm the merge to `next`. A follow-up series to add `objecttype` support is already planned.

---

### **`git history squash` lands as a rebase backend**
**Thread**: [PATCH v9] `git history squash`: fold a range into its oldest commit
**Author**: Harald Nordgren
**Status**: **Technically complete and queued for `next`**.

The **final iteration (v9)** addressed the last open question: **`amend!` message handling**. The command now incorporates `amend!` messages targeting the oldest folded commit into the default squashed message, resolving an internal inconsistency where validation and message construction diverged. The series also added **test helpers** to improve readability, following the pattern from the `delete-merged` topic.

### Key features

- **Autosquash alignment**: The `--reedit-message` template now **exactly matches `git rebase -i --autosquash`**, grouping `fixup!`/`squash!`/`amend!` commits under their targets.
- **Rev-list option validation**: Warns if user-provided options (e.g., `--reverse`) would alter the walk order, preventing silently incorrect results.
- **Ref-handling**: Rejects operations with interior refs by default, with advice to use `--update-refs=head`.

**Why it matters**: Junio C Hamano **fully endorsed** the vision of using `git history squash` and `git replay` as a **faster, non-interactive backend for `git rebase --autosquash`**, calling it "an ideal future." This series is a **foundational piece** of that effort, eliminating the repeated conflict resolution of the current rebase implementation. The architectural shift toward modular, conflict-free history editing is now a **project-level aspiration**.

**Next steps**: Integration into `next`, with follow-up work to close feature-compatibility gaps (e.g., conflict handling).

---

### **`git replay --linearize` nears resolution**
**Thread**: [PATCH v7] `git replay`: introduce `--linearize` option
**Author**: Toon Claes
**Status**: **Design debate resolved**; awaiting final patch.

The **multi-branch ambiguity**—where `--linearize` flattens all branches into a single history—sparked a **philosophical debate** between **flexibility (Junio)** and **strictness (Elijah Newren)**. The resolution:
- **Junio’s position**: The current behavior is **not a bug but a feature**. Users who want to avoid concatenation can invoke `git replay` multiple times, one branch at a time. He proposed **syntax alternatives** (e.g., limiting to a single negative ref plus positive refs) but stopped short of endorsing them.
- **Elijah’s counter**: The current behavior **violates the principle of least astonishment**, ignores command-line order, and creates a **recoverability asymmetry** (untangling concatenated branches is harder than concatenating independent ones). He proposed a **minimal fix**: restrict `--linearize` to a single positive ref, aligning with `--advance`/`--revert`.

**Outcome**: Toon Claes is expected to **adopt Elijah’s minimal fix** in the next iteration, unblocking the series for merging. The change preserves future flexibility while addressing the most common use case (linearizing a single branch).

**Why it matters**: `--linearize` is a **simpler alternative** to Johannes Schindelin’s earlier merge-replay implementation, offering predictable, all-or-nothing flattening. The resolution ensures the feature is **usable and consistent** with other `git replay` modes.

---

### **ODB abstraction series ready for `next`**
**Thread**: [PATCH v4 0/9] Introduce object filters to `odb_for_each_object()`
**Author**: Patrick Steinhardt
**Status**: **Queued for `next`** after Junio’s sign-off.

The **final iteration (v4)** fixed trivial commit-message typos (function name mismatches). The series introduces **object filters to `odb_for_each_object()`**, enabling backends to optimize traversals (e.g., using bitmaps) and simplifying `git cat-file --batch-all-objects`. Key changes:
- **New `filter` field** in `odb_for_each_object_options`, allowing backends to skip excluded objects.
- **Bitmap-based filtering** for the packed backend, with source-aware iteration to prevent misattribution.
- **Refactored `git cat-file`** to use ODB-mediated filtering (7-insertion/69-deletion diffstat).

**Why it matters**: This is a **major step toward pluggable ODBs**, allowing Git to support alternative storage systems (e.g., cloud-based object stores) without backend-specific hacks. The series is **performance-neutral** for existing workflows but enables future optimizations.

**Next steps**: Junio’s "What’s cooking" report will confirm the merge. A follow-up series to **split the callback type** for bitmapped object iteration is planned.

---

### **Rebase dropped-commit fixes graduate**
**Thread**: [PATCH v3 0/9] sequencer: avoid copying notes from dropped commits
**Author**: Phillip Wood
**Status**: **Queued for `next`** after addressing Junio’s feedback.

The **final iteration (v3)** fixed two behavioral regressions in `join_revs()` (missing `UNINTERESTING` propagation and redundant traversal avoidance) and updated documentation to reflect the new 64-branch limit. The series **systemically fixes** how the rebase machinery handles dropped commits, ensuring notes and post-rewrite hooks no longer receive spurious data.

### Key changes

- **`enum pick_result`** to distinguish dropped commits from successful picks.
- **Error handling** for external merge strategies and command execution failures.
- **Test coverage** for edge cases (conflict resolution, multiple fixups, missing pick commands).

**Why it matters**: This **eliminates a long-standing source of corruption** in rebase operations, particularly for users of `git notes` or `post-rewrite` hooks. The series is a **tested-level contribution** with no known regressions.

---

## In brief

- **`git fetch --submodule-errors`** (Paulius Zaleckas): **Ready for `next`** after addressing Junio’s mechanical feedback (array indexing type and naming). Adds `fetch.submoduleErrors` config and `--submodule-errors` CLI to control whether submodule fetch failures abort the parent fetch.
- **`git show-branch` slab conversion** (Gatla Vishweshwar Reddy): **Ready for `next`** after fixing behavioral regressions in `show_independent()` and `join_revs()`. Lifts the 27-branch limit to 64 using a configurable-stride commit-slab.
- **`gitweb` object ID rendering** (Travor Liu): **Under review**. Fixes a bug where index lines in commitdiff view showed full, unlinked object IDs instead of shortened, clickable hashes.
- **`git mv` misleading error** (Lucas Zamboni Orioli): **Under review**. Fixes a bug where `git mv` blamed the source file for a missing destination directory. Ben Knoble raised a **TOCTOU race** and suggested improving the error message.
- **Trace2 timestamp crash** (Derrick Stolee): **Under review**. Fixes a crash in trace2 telemetry when timestamp formatting fails, replacing `xsnprintf()` with `snprintf()` and adding defensive initialization.
- **`git rev-list --no-walk` path filtering regression** (Peter Colberg): **Root cause identified**. Kristofer Karlsson (author of the problematic commit) proposed a fix to restore `process_parents()` in the `REV_WALK_NO_WALK` case.

---

## On the radar

- **`git replay --linearize`**: Awaiting Toon Claes’s final patch with Elijah’s minimal fix.
- **`git subtree` safeguard** (Ian Jackson): Awaiting v2 with test style improvements.
- **`diff.<driver>.process`** (Michael Montalbo): **Technically complete** after v5, but Junio’s sign-off is pending. Enables external tools to inject diff hunks into Git’s pipeline.
- **`the_repository` removal**: Multiple series (`ps/refs-wo-the-repository`, `rs/tempfile-wo-the-repository`) are **queued for `next`**, continuing the project-wide effort to eliminate global state.

---

## Editorial note

Today’s traffic underscored Git’s **maturity as a platform**: long-running refactoring efforts (ODB abstraction, `the_repository` removal) are bearing fruit, while new features (`git history squash`, `git replay --linearize`) are being designed with **architectural foresight**. The **debate over `git replay --linearize`** was a rare example of **philosophical disagreement**—flexibility vs. strictness—resolved through **pragmatic compromise**. Meanwhile, **bugfixes** (trace2 crashes, `git rev-list` regressions) show the project’s **commitment to stability**, even in edge cases.