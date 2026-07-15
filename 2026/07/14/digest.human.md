# Here is the daily digest for the Git mailing list on 2026/07/14:

---

## The day in brief

A busy day on the Git mailing list, with **135 emails across 25 threads**. The most significant developments include:

- **`git cat-file --batch-command remote-object-info`** (Pablo Sabater) reached its **17th and final iteration**, addressing all prior feedback and resolving the last remaining bounds-checking concern. The series is now **ready for merging to `next`**.
- **`git branch --delete-merged`** (Harald Nordgren) landed in its **19th and final version**, completing a long-running effort to provide safe, automated local-branch cleanup with comprehensive filtering and safety controls. The series is **ready for `next`**.
- **`git history squash`** (Harald Nordgren) saw its **9th and final iteration**, with all prior feedback addressed and the series now **ready for integration**. Junio Hamano’s "Will replace" sign-off on v7 suggests it will likely be queued for the next release.
- **`git log --graph` visual root indentation** (Pablo Sabater) reached **v12**, with all mechanical feedback addressed and the series **ready for final review** before integration into `next`.
- **Packfile URI download races** (Ted Nyman) saw a **design pivot** after Junio Hamano and Jeff King raised concerns about the initial approach’s abandonment of resumable downloads. The author now plans to revise the design to preserve predictable partial-pack paths while preventing concurrent corruption.
- **`the_repository` removal efforts** continued with two series: Tian Yuchen’s **10-patch migration of config variables into `struct repo_config_values`** (v12, ready for `next`) and René Scharfe’s **5-patch introduction of repository-aware tempfile/lockfile APIs** (queued in `seen`).

The day also featured **user support requests**, **bug reports**, and **design discussions**, including a proposal for persistent shallow fetch options and a "fake-linear mainline" mechanism to reduce repository bloat in merge-heavy histories.

---

## Notable threads

### `git cat-file --batch-command remote-object-info` (Pablo Sabater)

The **17th and final iteration** of this security-hardened feature series is now **complete and ready for merging to `next`**. The series implements `remote-object-info` support for `git cat-file --batch-command`, allowing clients to query object metadata (currently just size) from remote repositories without downloading full objects. Key improvements in v17:

- **Bounds-checking in response parsing**: Added verification of `object_info_values.nr > 1 + size_index` before accessing the array, preventing out-of-bounds reads if a buggy or malicious server returns fewer values than expected.
- **Cover letter attribution**: Corrected the v16 cover letter to attribute the work to Pablo Sabater as the primary author.
- **Memory safety**: Fixed two memory leaks in the transport helper subsystem and reaffirmed the fix in v17.
- **Dynamic placeholder validation**: Implemented runtime capability-based filtering of format placeholders, directly addressing segfault and `die()` issues identified in earlier versions.
- **Test coverage**: Added **699 lines of new tests** covering all transport types, edge cases, and error conditions.

The series is now **technically complete**, with all prior feedback addressed and no remaining open questions. Junio Hamano’s administrative revert of v16 (due to the cover letter attribution mismatch) has been resolved, and the series is on track for `next`. The implementation balances security, compatibility, and maintainability, with strict input validation, protocol enforcement, and comprehensive test coverage.

---

### `git branch --delete-merged` (Harald Nordgren)

The **19th and final version** of this series landed, completing a long-running effort to provide safe, automated local-branch cleanup with comprehensive filtering and safety controls. The series introduces a new `--delete-merged` subcommand that safely deletes local branches whose work has already landed on their upstream, while preserving branches that are still needed (e.g., checked-out, used as upstreams for unmerged branches, or opted out via config). Key features:

- **Repeatable upstream selector**: The `--delete-merged` option can now be specified multiple times (e.g., `--delete-merged origin/main --delete-merged origin/next`) to widen the upstream match, with optional positional patterns (e.g., `topic-*`) to limit deletion scope.
- **Refined stacked-branch protection**: If a branch is used as an upstream for an unmerged branch, it is spared, and its own stale upstream config is cleared if the upstream is being deleted. This avoids broken dependency chains while cleaning up stale configs.
- **Per-branch opt-out**: Users can exempt specific branches from automated cleanup via `branch.<name>.deleteMerged=false`, while still allowing explicit deletion via `git branch -d`.
- **`--dry-run` preview**: Prints which branches would be deleted without modifying refs or config.
- **Test coverage**: Comprehensive integration tests verify all safety checks, edge cases (e.g., self-referential upstreams, missing upstreams), and the new repeatable upstream selector behavior.

The series is **ready for `next`**, with all prior feedback addressed and no remaining open questions. The only agreed follow-up is extending the refined stacked-branch protection to `git branch -d` for consistency, which will be implemented after the series lands. The implementation is well-motivated, thoroughly tested, and aligns with Git’s API consistency expectations.

---

### `git history squash` (Harald Nordgren)

The **9th and final iteration** of this series is now **ready for integration**, with all prior feedback addressed. The series implements `git history squash`, a new command that efficiently collapses a linear or merge-commit-containing range into its oldest commit while preserving descendant history. Key improvements in v9:

- **`amend!` message handling**: Incorporated `amend!` messages targeting the first folded commit into the default squashed commit message, addressing Matt Hunter’s consistency concern.
- **Test suite readability**: Introduced test helpers to clarify the test suite, following the pattern established in the `delete-merged` topic.
- **Input validation**: Sanitized all `rev-list` options to ensure the walk order is not altered by user-provided options, preventing silently incorrect results.

The series is **technically complete**, with Junio Hamano’s "Will replace" sign-off on v7 indicating intent to merge. The only open question—whether `--reedit-message` (or `--edit`) should be the default—remains unresolved but is not a blocker. The implementation is well-tested, with clear documentation and a flags-based API designed for future extensibility. The series is now awaiting Junio’s next "What's cooking" report for formal queuing.

---

### `git log --graph` visual root indentation (Pablo Sabater)

The **12th iteration** of this feature series is now **mechanically finalized** and **ready for final review** before integration into `next`. The series introduces indentation in `git log --graph` to clearly represent commits with excluded parents by progressively indenting visual roots (except the first) while preserving parent-child relationships. Key features:

- **Lookahead buffer**: Safely determines visual root status during revision traversal.
- **Four-level wrap**: Prevents symmetry-induced ambiguity in cascading roots.
- **Opt-out mechanism**: Allows disabling indentation via `--[no-]graph-indent` or `log.graphIndent`, with indentation enabled by default.
- **Test coverage**: New test file (`t4218-log-graph-indentation.sh`) covers the four-level wrap, opt-out behavior, and edge cases.

The series is **technically complete**, with all mechanical feedback addressed in v12. Junio Hamano and Phillip Wood are the remaining reviewers; their sign-off is the last blocker. The implementation is well-structured, with clear separation between the lookahead buffer, visualization logic, and opt-out mechanism. The **redundant-indentation edge case** (marked NEEDSWORK in v8) remains documented but unaddressed, and follow-up work includes reconciliation with Kristofer Haugsbakk’s `kk/prio-queue-get-put-fusion` topic and cleanup of `get_commit_action()` side effects.

---

### Packfile URI download races (Ted Nyman)

This bugfix series saw a **design pivot** after Junio Hamano and Jeff King raised concerns about the initial approach’s abandonment of resumable downloads for packfile URIs. The initial design used unique tempfiles for each packfile URI download, preventing corruption but sacrificing resumability. Key developments:

- **Junio’s concern**: Resumability is a major goal of the packfile URI feature, and the proposed fix deliberately sacrifices it.
- **Jeff King’s alternative**: Suggested treating the `.pack.temp` file as a lockfile, borrowing Git’s existing lockfile logic to avoid corruption while preserving resumption.
- **Ted Nyman’s response**: Acknowledged the trade-off was premature and plans to revise the design to preserve predictable partial-pack paths while preventing concurrent corruption. The reroll will address handoff scenarios (where one process takes over a stalled download) and stale-file cleanup (leftover `.pack.temp` files from crashed processes), and will unify the solution for both packfile URIs and dumb HTTP.

The thread’s focus has shifted from a "fix first, optimize later" approach to a more nuanced solution that balances correctness, coordination complexity, and user expectations. The reroll is expected to include mechanical improvements to `fetch-pack` parsing logic (adopting Jeff King’s suggested `strbuf` refactor) and a correction to the `--index-pack-args` documentation typo.

---

### `the_repository` removal efforts

Two series advanced the ongoing effort to eliminate implicit reliance on the global `the_repository`:

1. **Tian Yuchen’s 10-patch series (v12)** migrates global configuration variables into `struct repo_config_values`. The series is now **complete and ready for `next`**, with all prior feedback addressed, including a style fix for the declaration-after-statement warning in `apply.c`. The series touches `environment.c`/`environment.h` (primary), `repository.c`, `repository.h`, and various subsystem-specific files. Key improvements:
   - **Destructor extensibility**: The `repo_config_values_clear()` function is designed to be extended as more variables are migrated.
   - **Memory management**: All heap-allocated fields are properly freed during re-assignment and teardown.
   - **Circular dependency resolution**: The `push_default` enum is temporarily kept in `environment.h` to avoid circular header dependencies, with a `NEEDSWORK` comment flagging this as a future cleanup opportunity.

2. **René Scharfe’s 5-patch series** introduces repository-aware variants of the tempfile and lockfile APIs (`repo_create_tempfile`, `repo_hold_lock_file_for_update`, etc.). The series is **queued in Junio’s `seen` branch**, with no glaring issues identified in his cursory review. The patches are mechanical and confined to plumbing, with no expected user-visible behavior changes. The series touches 21 files across apply, builtins, ref backends, commit-graph, midx, ODB, and config.

Both series are incremental steps in the broader `the_repository` removal effort, which aims to make Git’s codebase more modular and thread-safe.

---

## In brief

- **`git show-branch` slab conversion** (Gatla Vishweshwar Reddy): Junio Hamano and Jeff King identified conceptual gaps in the v2 patch’s fixed-width slab design. The author will need to address these points before the patch can advance.
- **`git fetch --submodule-errors`** (Paulius Zaleckas): Junio C Hamano requested mechanical refactoring of config value parsing and option forwarding logic to improve maintainability and consistency. The series remains under review.
- **`git fast-export` help text alignment** (Christian Couder): Queued in Junio’s `seen` branch after a surface-level review from Patrick Steinhardt.
- **Test modernization for `t1100-commit-tree-options.sh`** (Shlok Kulshreshtha): Queued for `next` after addressing all prior feedback. The series updates the script to use modern Git test style conventions.
- **Packfile URI download races** (Ted Nyman): Design pivot underway to preserve resumable downloads while preventing concurrent corruption. The reroll will address handoff scenarios, stale-file cleanup, and dumb HTTP unification.
- **`strbuf_getwholeline()` refactoring** (René Scharfe): Queued in `next` after Junio C Hamano’s review. The patch removes a redundant `strbuf_reset()` call in the `HAVE_GETDELIM` code path.
- **`git last-modified` performance inquiry** (Gusted): A Forgejo contributor reported that `git last-modified -z -t --max-depth=0` is 2.5× slower than an equivalent `git log` pipeline. No resolution yet.
- **Git for Windows 2.55.0(3) release** (Johannes Schindelin): Announced a bugfix release addressing heap overflows in the `wincred` credential helper and updating Git Credential Manager to v2.9.0.
- **Persistent shallow fetch and fake-linear mainline** (Richard Fine): Proposed two Git enhancements to address pain points in large repositories with long-running branches and merge-heavy histories. The proposals are in the design discussion phase.
- **Coverity unchecked return values** (Johannes Schindelin): An 11-patch series addressing unchecked return values flagged by Coverity. The patches are small, focused fixes that add error checking to API calls that previously ignored their return codes. The series is under review.

---

## On the radar

- **`git log --cherry-mark` bug** (Uwe Kleine-König): Reported a bug where `git log --cherry-mark --right-only` fails to mark equivalent commits when the revision range includes a merge commit. No patch yet posted.
- **`git last-modified` performance** (Gusted): The performance gap between `git last-modified` and a hand-rolled `git log` pipeline remains unexplained. Further investigation is needed.
- **Persistent shallow fetch and fake-linear mainline** (Richard Fine): The proposals for persistent shallow fetch options and a "fake-linear mainline" mechanism are in the design discussion phase. Feedback is needed on whether the problems are widespread enough to justify new features and how to address edge cases (e.g., reachability guarantees, `git gc` behavior).