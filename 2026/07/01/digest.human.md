# The Git Mailing List Daily Digest for 2026/07/01

**The day in brief.** Wednesday, July 1, 2026 saw 134 emails across 29 threads. The day was **heavy on finalized feature work and post-merge follow-ups**, with several long-running series reaching resolution. The **`git history drop`** and **`git replay --linearize`** features were both accepted for merging after critical disputes were resolved, while **`git cat-file --batch-command` remote object metadata support** reached its 15th iteration with all prior feedback addressed. A **Coverity-driven cleanup series** dominated the "In brief" section, and **Git for Windows** issues continued to surface in platform-specific threads.

---

## Notable threads

### `git cat-file --batch-command` remote object metadata support reaches v15
**Pablo Sabater**’s GSoC project to add `remote-object-info` support to `git cat-file --batch-command` is now **ready for merging** after 15 iterations. The series allows clients to query object metadata (initially just size) from remote repositories without downloading full objects, using a new `remote-object-info` command that works with protocol v2 servers advertising the `object-info` capability.

### Key improvements in v15:

- Replaced `strtoul_szt()` with `strtoumax_szt()` for platform-independent `size_t` handling.
- Added **dynamic capability-based format placeholder validation**, ensuring only supported atoms (e.g., `%(objectname)`, `%(objectsize)`) are accepted.
- **Security hardening**: 10,000-object batch limit, strict protocol v2 enforcement, and silent continuation for unsupported fields (matching `for-each-ref` behavior).
- **680 lines of new tests** in `t/t1017-cat-file-remote-object-info.sh`.

**Critical issue resolved:** Junio Hamano identified a **refactoring flaw in patch 5/13** (uninitialized local variable and lingering global variable in `write_fetch_command_and_capabilities()`). The fix was straightforward: initialize the local variable to `0` and split the unrelated `hash_algo` type change into a separate patch.

**Protocol extensibility consensus:** Pablo documented that future metadata features (e.g., `objecttype`) will be appended to the same space-separated list in the `object-info` capability value (e.g., `object-info=size type`). The series is **technically complete** and carries **Tested-by from Uwe Kleine-König**.

---

### `git history drop` subcommand accepted for merging
**Patrick Steinhardt**’s 11-patch series introducing the `git history drop` subcommand was **accepted by Junio Hamano** after resolving a **critical ref resolution dispute** in v8. The command removes a commit from history and replays its descendants on top of its parent, with conflict detection, bare repository support, and preservation of local changes.

### Key features:

- **Conflict detection**: Aborts if replaying descendants would result in conflicts or overwrite local changes.
- **Bare repository support**: Works in both bare and non-bare repositories.
- **Ref updates**: Moves branches pointing to the dropped commit to its parent, with configurable scope via `--update-refs=(branches|head)`.
- **Dry-run mode**: Supports `--dry-run` to preview ref updates.
- **561 lines of test coverage** in `t/t3454-history-drop.sh`.

**Critical dispute resolved:** Junio identified a **logical flaw in v7** where `find_head_tree_change()` searched for symbolic branch names in `result->updates[]` while `compute_pending_ref_updates()` filtered the array to contain only `HEAD` under `--update-refs=head`. The v8 fix adds `RESOLVE_REF_READING` to `refs_resolve_ref_unsafe()`, ensuring it returns `NULL` when `HEAD` cannot be resolved.

**Reset API modernization:** The series also modernizes the reset API, renaming `reset_head()` to `reset_working_tree()`, converting flags to an enum, adding dry-run mode, and making HEAD updates opt-in. The changes are **backward-compatible** and advance the `the_repository` removal effort.

---

### `git replay --linearize` merged with post-merge issues identified
**Toon Claes**’s series adding `--linearize` to `git replay` was **merged to `master`**, but **three critical issues** were identified post-merge:
1. **Silent commit dropping regression** in single-branch replay with merge commits (Johannes Schindelin, **highest priority**).
2. **CLI design inconsistency** with `git rebase` (Patrick Steinhardt, **strategic concern**).
3. **Merge commit divergence handling** (Junio Hamano, Phillip Wood, **design limitation acknowledged**).

**Root cause of regression:** v5’s base-selection logic **inadvertently removed the `replayed_base` mechanism**, causing only the tip commit to be replayed while intermediate commits (including merges) are lost. Johannes provided a test case (`master~2..master`) showing v5 replays only the tip commit directly onto `--onto`, dropping the "Git 2.55-rc2" commit entirely.

**Design intent clarified:** Toon confirmed `--linearize` is intended to produce a **single linear sequence** regardless of input branches, even if this duplicates shared history. This aligns with Johannes’s preference for predictable behavior but diverges from `git rebase --no-rebase-merges`.

### Follow-up required:

- **Urgent patch** to restore `replayed_base` logic or redesign multi-branch handling.
- **Documentation update** to clarify `--linearize` behavior.
- **Test cases** to expose merge commit divergence behavior.
- **CLI design decision**: Adopt `git rebase` syntax, diverge with clearer UX (e.g., `--replay-merges=<mode>`), or hybrid approach.

---

### `git history squash` series converges on template design
**Harald Nordgren**’s series adding a `squash` subcommand to `git history` reached consensus on the **commit-message template format** after Junio Hamano and Phillip Wood challenged the initial divergence from `git rebase -i`. The final design:
- **Numbered list of commit subjects** (for context).
- **Editable message body** with `squash!` bodies retained (separated by a blank line from their base commit’s message).
- **`fixup!`/`amend!` messages omitted** to reduce visual clutter.

**Key debate:** Phillip Wood successfully argued that `git history` is an **explicit UX experimentation space** and provided **concrete evidence of a usability problem with `git rebase -i`’s current template** (excessive commented-out lines forcing users to scroll past irrelevant noise). Junio’s challenge ("what is wrong with the current format?") was resolved by this evidence.

### Open questions:

- Should `--reedit-message` (or `--edit`) be the default? Phillip advocates for this as a **commit hygiene measure**.
- **Recoverability concerns**: Matt Hunter and Phillip Wood noted that `git reset --hard` is insufficient to undo operations where `--update-refs` moves multiple branches, as Git’s reflog lacks visibility into which refs were affected. Phillip suggested **reflog transaction IDs** or a separate operations log as a potential solution.

---

### Coverity-driven cleanup series dominates "In brief"
**Johannes Schindelin**’s 13-patch series addressing Coverity-flagged resource leaks and error-path bugs was **queued by Junio Hamano** despite unresolved correctness concerns in two patches:
1. **Patch 1/13**: Junio identified a **critical flaw** in the `errno`-based error detection, which may be clobbered by intervening function calls (`strbuf_release`, `insert_loose_map`). He recommended using `ferror(fp)` instead.
2. **Patch 5/13**: Patrick Steinhardt pointed out that the `dpath` variable is initialized to `NULL` but never assigned a non-`NULL` value before the `free(dpath)` call, rendering the fix ineffective.

### Key fixes:

- **`bloom.c`**: Slab initialization leak (idempotent flag).
- **`revision.c`**: Bloom-filter keyvec leak (explicit freeing).
- **`line-log.c`**: Redundant range copy leak.
- **`dir.c`**: Untracked-cache data leaks.
- **Windows**: Process-handle leak and double-close bug.

The series is **self-contained and uncontroversial** aside from the two patches, which will likely be addressed in follow-ups.

---

## In brief

> **`git repo` GSoC project** -- K Jayatheerth’s weekly update (week 5) continues the research/design phase, with no on-list technical discussion or patches. The project remains in its initial phase, with updates communicated via external blog posts.

> **`git cat-file --remote-object-info` GSoC project** -- Pablo Sabater’s weekly update (weeks 4–5) reports progress on extending `git cat-file --batch-command` with remote object metadata support. No technical details shared on-list.

> **Reftable quadratic-time behavior fix** -- Kristofer Karlsson’s patch exposes tombstones to iterator bounds checks, reducing runtime from **O(n²) to O(n)** (e.g., 8,000 refs: 14.93s → 0.145s). The fix was **merged after Patrick Steinhardt’s review**, which acknowledged the trade-off of added code complexity at call sites.

> **`excludes_file` migration into `repo_config_values`** -- Tian Yuchen’s series completed the **three-phase guardrail migration** (silent return → `BUG()` → no check) and was **merged into `next`**. The series eliminates the global `excludes_file` variable as part of the libification effort.

> **`git blame -b` output formatting fix** -- René Scharfe’s patch removes the extra hex digit reserved for an unused caret marker, aligning abbreviated commit hashes with `core.abbrev`. The fix was **confirmed by the original reporter (Laszlo Ersek)** and is ready for merging.

> **HTTP/HTTPS authentication regression in Git for Windows** -- A user reported that `http."<url>".allowNTLMAuth true` no longer works in Git for Windows 2.55.0.windows.1. Johannes Schindelin redirected the discussion to the Git for Windows issue tracker ([#6308](https://github.com/git-for-windows/git/issues/6308)).

> **`git gui` encoding mismatch fix** -- A patch fixes `git gui` failures on Windows when the home directory contains non-ASCII characters (e.g., EntraID/AzureAD profiles). The fix replaces `safe_exec` with `safe_open_command -encoding utf-8` to match `cygpath`’s UTF-8 output.

> **Meson build race fix** -- D. Ben Knoble’s patch restores `hook-list.h` to the `builtin_sources` list in `meson.build`, preventing a race condition that causes build failures when the header is missing.

> **CI PID limits for private GitHub repositories** -- Johannes Schindelin’s patch adjusts Dockerized CI jobs in private repositories to use explicit process/file limits (`--pids-limit=16384`, `--ulimit=nproc=16384`, `--ulimit=nofile=32768`), preventing resource exhaustion.

> **Bloom filter leak fixes** -- Jeff King’s 3-patch series plugs memory leaks in Bloom-filter code paths exposed by `GIT_TEST_COMMIT_GRAPH_CHANGED_PATHS=1`. The series was **endorsed by Junio Hamano** and is ready for merging.

---

## On the radar
- **`git history` reorganization into `lib/` directory** -- Patrick Steinhardt’s RFC v3 series remains stalled due to **skepticism about the discoverability rationale** (Phillip Wood, Junio Hamano). The debate has shifted toward whether the structural benefits (e.g., enforcing coding conventions) justify the disruption.
- **`git rebase` dropped-commit notes fix** -- Phillip Wood’s 11-patch series is **queued in Junio’s tree** but carries a **known limitation**: the "edit" command still erroneously records dropped commits as rewritten. A follow-up patch is expected.
- **`git merge-base` optimization** -- Tian Yuchen’s series optimizing `paint_down_to_common()` for one-sided histories is **ready for merging** after a procedural rebase on `kk/commit-reach-find-all-fix`. The series delivers **100-1000x speedups** for asymmetric queries.
- **`git format-patch` leak fix** -- Jeff King’s 2-patch series plugs a memory leak in `git format-patch --base` and improves test suite leak reporting. The series is **ready for merging** and includes a **CI infrastructure proposal** to consolidate leak-checking jobs.