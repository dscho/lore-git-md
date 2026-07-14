# The day in brief

**2026-07-13** was a **heavy-traffic day** (130 emails, 34 active threads) with **three major series reaching final review or integration**, a **contentious design debate** in `git replay --linearize`, and **multiple bugfixes and refactorings** landing or advancing. The standout developments:

- **`git log --graph` visual root indentation (v11)** by Pablo Sabater is **technically complete and ready for `next`**, with all design decisions resolved and only a minor mechanical tweak pending.
- **`git branch --delete-merged` (v18)** by Harald Nordgren is **implementation-complete and awaiting final review**, with all safety mechanisms (stacked-branch protection, per-branch opt-out, `--dry-run`) now in place.
- **ODB abstraction work** by Patrick Steinhardt saw **two series (`ps/odb-pluggable-housekeeping` and `ps/odb-for-each-object-filter`) approved for `next`**, marking significant progress toward pluggable backends.
- **`git replay --linearize`** sparked a **principle-of-least-astonishment debate** between Elijah Newren and Junio C Hamano over its multi-branch behavior, with no resolution yet.
- **Bugfixes** for packfile URI races, reftable tombstone performance, and sequencer dropped-commit notes were posted or advanced.

---

## Notable threads

### `git log --graph` visual root indentation (v11) -- **ready for `next`**
**Author**: Pablo Sabater
**Status**: Technically complete; all design decisions resolved (lookahead buffer, four-level wrap, opt-out mechanism). Junio Hamano and Phillip Wood have not yet signed off on the final patches, but all prior feedback has been addressed. The series is **ready for `next`** once the minor mechanical tweak (bit-packing booleans) is incorporated.

The series introduces indentation for "visual roots" (commits with excluded parents) in `git log --graph` to clarify ancestry relationships. Key features:
- **Lookahead buffer** to safely determine visual root status during revision traversal.
- **Four-level wrapping** to prevent excessive indentation.
- **Opt-out mechanism** (`--[no-]graph-indent` flag and `log.graphIndent` config) for users who prefer the traditional layout.
- **Extensive test coverage** for edge cases (single roots, cascading roots, merge parents, boundary commits, filtering options).

The only remaining action is Junio’s **readability tweak** in patch 7/7 (replacing `revs->graph_indent_set > 0` with `revs->graph_indent_set`), which Pablo Sabater has acknowledged. The series is **uncontroversial** and represents a **significant usability improvement** for complex histories.

---

### `git branch --delete-merged` (v18) -- **implementation-complete**
**Author**: Harald Nordgren
**Status**: All planned features and safety mechanisms are implemented, including:
- **Stacked-branch protection (abort-and-clear)**: Fails to delete a branch if it’s used as an upstream for an unmerged branch, but clears stale upstream configs for kept branches.
- **Per-branch opt-out**: `branch.<name>.deleteMerged=false` exempts specific branches from automated cleanup.
- **`--dry-run` preview**: Shows what would be deleted without modifying refs.
- **`<branch>` argument support**: Limits deletion candidates using the `--forked` filter machinery.

Phillip Wood’s **final review** of the `--dry-run` patch (7/7) was **surface-level**, focusing on commit message wording and the `--dry-run --quiet` edge case. Harald and Phillip agreed to **reject the combination as nonsensical** (since `--quiet` suppresses output). The series is **ready for final review** and likely **ready for `next`** once the mechanical flags cleanup (agreed in earlier patches) is applied.

A **forward-looking discussion** about refining the `--forked` filter’s logic (to better distinguish true forks from long-lived branches) was noted but deferred as a follow-up.

---

### ODB abstraction: `ps/odb-pluggable-housekeeping` and `ps/odb-for-each-object-filter` -- **approved for `next`**
**Author**: Patrick Steinhardt
**Status**: Both series are **technically complete and approved** by Junio C Hamano and Patrick Steinhardt (ODB subsystem owner). They are now **queued in `next`** for integration testing.

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

Both series are **prerequisites for pluggable ODB backends** and represent **major architectural progress**. No substantive objections remain.

---

### `git replay --linearize` -- **design debate over multi-branch behavior**
**Author**: Toon Claes
**Status**: The series is **queued in `next`**, but Elijah Newren has raised a **principle-of-least-astonishment objection** to the `--linearize` option’s behavior when multiple positive refs are provided (e.g., `git replay --linearize --onto main topic1 topic2`). The current implementation flattens all commits into a single linear history, concatenating branches in revision-walk order, which Elijah argues is inconsistent with other `git replay` modes (`--advance` and `--revert`) that reject multiple positive refs.

### Key positions

- **Elijah Newren**: Proposes either (a) disallowing multiple positive refs with `--linearize` (aligning with `--advance`/`--revert`) or (b) tracking a `last_commit` per branch to preserve branch independence.
- **Junio C Hamano**: Pushes back, arguing that:
  - Unpredictable order is inherent to linearization (even with a single branch containing merges).
  - The current behavior is acceptable for cases where order is inconsequential (e.g., octopus merges of independent branches).
  - The escape hatch (using separate invocations for independent branches) is a feature, not a limitation.

The discussion is now about **design philosophy** (flexibility vs. strictness) rather than technical correctness. Toon Claes has not yet responded, and the series remains in `next` pending resolution. This could delay graduation to `master` if the ambiguity isn’t addressed.

---

### Sequencer: `pw/rebase-drop-notes-with-commit` -- **merged to `master`**
**Author**: Phillip Wood
**Status**: The 10-patch series fixing how `git rebase` handles notes from dropped commits is **now merged to `master`**. The series:
- Introduces `enum pick_result` to clarify the outcome of picking a commit.
- Prevents notes from being copied from dropped commits to the current HEAD.
- Handles edge cases like external merge strategy failures and command execution failures.
- Expands test coverage for both "apply" and "merge" backends.

Junio C Hamano’s **final review** confirmed the series is **technically sound and ready for integration**. This is a **significant bugfix** for the rebase machinery.

---

### Reftable: `kk/reftable-tombstone-quadratic-fix` -- **merged to `master`**
**Author**: Kristofer Karlsson
**Status**: The 2-patch series fixing a quadratic performance regression in the reftable backend is **now merged to `master`**. The fix:
- Moves tombstone suppression logic from the merged iterator to `struct reftable_stack_options`.
- Enables early termination when tombstones exceed the relevant bound.
- Improves performance from ~14s to ~0.2s in the perf test’s 8000-ref scenarios.

Patrick Steinhardt will update libgit2 to use the new `reftable_stack_options` field after both this series and `ps/reftable-hardening` graduate to `master`. The `suppress_deletions` flag will eventually be deprecated and removed.

---

## In brief

### Bugfixes
- **Packfile URI races**: Ted Nyman posted a 2-patch series fixing race conditions in Git’s HTTP transport when multiple processes fetch the same packfile URI. The patches:
  - Use unique temporary files for packfile URI downloads to prevent corruption.
  - Make `fetch-pack` accept both `"pack"` and `"keep"` success messages from `index-pack`.
  - Add regression tests for concurrent downloads and `.keep` file races.
- **`git rebase --autosquash` empty commits**: Farid Zakaria’s v3 patch (making `--autosquash` respect `--empty` for emptied commits) is **ready for integration** once its dependency (`pw/rebase-drop-notes-with-commit`) graduates to `next`. Phillip Wood expects the dependency to land shortly.
- **Cygwin IPv6 SCP URLs**: Ramsay Jones’s patch fixing a 5-year-old regression in IPv6-style SCP URL handling is **queued in `seen`**. The fix adds Cygwin-specific path validation to avoid misinterpreting `[` as a Windows drive letter.

### Refactoring and cleanup
- **`the_repository` removal**: Tian Yuchen’s v11 series migrating global config variables into `struct repo_config_values` is **ready for `next`**. The series:
  - Introduces `repo_config_values_clear()` to manage heap-allocated strings.
  - Migrates editor/pager/askpass programs, apply whitespace settings, and enums (`push_default`, `autorebase`, `object_creation_mode`).
  - Addresses all prior feedback, including memory leaks and redundant calls.
- **Test modernization**: Shlok Kulshreshtha’s 2-patch series modernizing `t/t1100-commit-tree-options.sh` is **ready for `next`** once a minor commit message tweak (present tense) is applied. The patches:
  - Replace backslash-continued `test_expect_success` blocks with modern quoted-body form.
  - Relocate a here-doc into the setup test to ensure it runs under the test harness.
- **`git fast-export` help text**: Christian Couder’s patch aligning the `git fast-export -h` usage string with the man page and Git conventions is **queued in `seen`**.

### Documentation
- **Contributor guidance**: Junio C Hamano’s 6-patch series clarifying contributor guidance in `Documentation/MyFirstContribution.adoc` and `Documentation/SubmittingPatches` is **under review**. The series:
  - Formalizes commit message structure (**Observation**, **Design**, **Implementation**).
  - Clarifies the role of the `seen` branch, `Reviewed-by:`/`Acked-by:` trailers, and "What's cooking" report style.
  - Addresses recurring questions about patch submission and review.

---

## On the radar

- **`git checkout --track=fetch`**: Harald Nordgren’s series is **stalled pending further review**. The author claims all feedback has been addressed, but Junio’s "What's cooking" report still lists it as "Waiting for response(s)."
- **`git history` subcommands**: The `hn/history-squash` and `ps/history-drop` topics are **cooking in `next`** but may face UI design debate. The new subcommands (`squash` and `drop`) are experimental and could spark discussion about command-line UI consistency.
- **`diff.<driver>.process`**: Michael Montalbo’s stalled series introducing external hunk providers for diff drivers is **expected to reroll within the week**. The feature allows delegating hunk detection to long-running external processes while letting Git handle output formatting.