# Git mailing list daily digest for 2026/08/14

## The day in brief
The Git mailing list saw **active design discussions** on source tree reorganization, ODB abstraction, and filesystem-level optimizations for `git worktree`. **Key patches advanced**: Siddharth Shrimali’s `git repack --drop-filtered` series was proposed for `next`, and Junio queued a performance regression fix for packfile handling. **New proposals** included Michael Montalbo’s data-driven approach to source tree reorganization and Peter Morris’s RFC for copy-on-write optimizations in `git worktree add`.

## Notable threads

### Source tree reorganization: data-driven counterproposal
**[2026/04/16/13-24-29]**
Michael Montalbo proposed a **data-driven methodology** to identify cohesive subsystems for incremental reorganization, addressing Junio C Hamano’s critique of Patrick Steinhardt’s `lib/` directory approach. The proposal uses **dual-signal analysis** combining commit history patterns (weighted by commit size) and call-graph relationships (derived from `cscope`) to group 163 of Git’s 231 `libgit.a` source files into 14 directories (e.g., `transport/`, `index/`, `revision/`). Boundary cases (e.g., `shallow.c` grouped with `transport/` despite its `revision:` commit label) are flagged as review targets.

The script’s innovation lies in its **mechanical, reproducible methodology**, avoiding the arbitrariness of the `lib/` approach while enabling incremental progress. The proposal reframes the debate around **how** to reorganize, not **whether**, and could break the deadlock if the community finds the methodology compelling.

### MIDX incremental writes: test coverage discussion
**[2026/06/12/20-07-05]**
Patrick Steinhardt reiterated a preference for **direct verification** that the MIDX contains objects from *all* layers (not just the newest one) when using `--base=none`, calling the current bitmap-based test "a bit roundabout." The exchange leaves the test-coverage question unresolved, though the core fix remains uncontested.

### `uploadpack.lazyFetchTrusted`: procedural and design feedback
**[2026/07/10/08-51-34]**
Christian Couder explained the misthreaded v2 patches: the `--in-reply-to` option was passed to `git format-patch` instead of `git send-email`, causing it to apply to every patch rather than just the cover letter. Junio C Hamano confirmed the diagnosis and suggested using `--no-chain-reply-to` for future resends.

Junio’s substantive reviews raised:
- A **stylistic improvement** to `lazy_fetch_objects()`: replace nested `if`/`return` with a single `return` using logical OR.
- A **scalability concern** about holding the list of missing object names in memory, sketching an alternative streaming API.
- **API design critiques** for `path_allowlist_apply()`, proposing a callback-based alternative to avoid hardcoding path validation logic.
- **Documentation ambiguities** in `uploadpack.lazyFetchTrusted`, including whether it is multi-valued and the syntax for specifying repositories.

### `git repack --drop-filtered`: proposed for `next`
**[2026/07/16/13-28-41]**
Siddharth Shrimali’s six-patch series introducing `git repack --drop-filtered` for partial clones was **proposed for `next`** after addressing all prior feedback. The series extends `git repack` with `--drop-filtered` and `--dry-run` options to safely reclaim disk space by removing locally cached promisor blobs exceeding a user-specified size threshold. Christian Couder reviewed v5 and stated, "This version looks good to me," and Junio queried whether to mark the topic for `next`. The author confirmed readiness, signaling the series is ready for integration testing.

### `git fetch_if_missing` refactoring: mechanically complete
**[2026/07/15/01-18-50]**
Tian Yuchen’s v5 series moving the global `fetch_if_missing` variable into `struct repository` is now **mechanically complete**, eliminating all remaining direct uses of `the_repository->fetch_if_missing`. Junio confirmed the series is "Much better" but did not signal whether it will be queued. The broader architectural question of whether `fetch_if_missing` belongs in `struct repository` remains unresolved.

### ODB alternates: design discussion on case-sensitivity
**[2026/08/07/11-56-23]**
Jeff King (Peff) and Junio C Hamano engaged in a **substantive design discussion** about the case-sensitivity handling in Patrick Steinhardt’s ODB alternates refactoring series. Peff argued that `core.ignoreCase` is a flawed proxy for alternate path comparisons, as it reflects the repository’s filesystem, not the alternate’s. He proposed simplifying the patch by **dropping case-insensitivity entirely**, accepting occasional duplicates as a low-risk trade-off. Junio endorsed this preference but raised concerns about `stat()`-based deduplication due to platform-specific filesystem behaviors (e.g., macOS Unicode normalization). The discussion solidified consensus toward simplicity, though no revised implementation has been posted.

### Packfile performance regression: merged to `next`
**[2026/08/12/19-11-09]**
Junio C Hamano queued Johannes Schindelin’s patch fixing a **quadratic-time performance regression** in `packfile_store_add_pack()` for the `next` branch. The patch introduces a fast-path optimization to skip duplicate checks for caller-guaranteed new packfiles, eliminating the O(N²) scan. Junio’s maintainability concerns about the `skip_dup_check` parameter remain unresolved but are deferred for post-merge follow-up. Jeff King revealed that Git already maintains a hashmap for tracking packfiles, reframing the fast-path as a targeted optimization to avoid the O(N) scan even when uniqueness is already guaranteed.

### Filesystem-level copy-on-write for `git worktree add`
**[2026/08/14/10-40-30]**
Peter Morris proposed leveraging **filesystem-level copy-on-write (CoW)** or block cloning (e.g., ReFS, Btrfs, APFS) for `git worktree add` to avoid physically duplicating data until modification. The RFC targets scenarios like concurrent AI coding agents, where multiple worktrees are created for the same repository. Junio C Hamano outlined the core architectural challenge: the checkout machinery (`checkout_entry()`) lacks visibility into sibling worktrees. He proposed intercepting `checkout_entry()` to perform CoW based on a mapping of blob objects to cleanly checked-out files across all worktrees, while flagging TOCTOU races and side effects on other commands. The proposal is architecturally feasible but high-effort, with implications for correctness, performance, and cross-platform consistency.

### `git rev-parse`: compute object names in other algorithms
**[2026/08/14/13-48-04]**
Dimitri John Ledkov introduced `repo_oid_to_algop()` to compute object names in a different hash algorithm on demand, enabling interoperability between repositories using different object formats (e.g., SHA-1 and SHA-256). The feature is narrowly scoped: it does not convert commits or persist computed names but recursively converts trees, tags, and blobs, rejecting submodules to avoid unbounded traversal. The patch includes an in-memory cache (`compat_oid_cache`) to avoid redundant computation and test coverage for edge cases.

### Submodule: warn on valueless `submodule.active` config
**[2026/08/14/17-37-29]**
Tilak Raaz (GSoC microproject) proposed warning on valueless `submodule.active` configuration entries using `repo_config_get_value_multi()` and `config_error_nonbool()`. Junio C Hamano questioned whether the command should **fail** (exit non-zero) or **continue** (emit warning only) after encountering a valueless key, noting that continuing may not be sensible for broken configuration. The test portion of the patch remains whitespace-damaged, delaying substantive review.

## In brief
- **[2026/08/04/19-56-32]** Vincent Mailhol and Kristoffer Haugsbakk discussed a cover letter formatting glitch in the `git history` Bash completion series, where a line starting with `#3` was misinterpreted as a comment by `b4`. No reroll is needed.
- **[2026/08/07/10-45-06]** Kristoffer Haugsbakk identified the root cause of a CI failure in the documentation series: the patch used a code block (`----`) where other synopsis-style documentation uses an open block (`--`). A v4 will address this.
- **[2026/08/12/06-42-37]** Elijah Newren’s patch updating the short help message for `git diff -l` was marked for inclusion in `next`, correcting the misleading implication that the option disables all rename detection.
- **[2026/08/14/18-55-36]** Matthew Hughes reported a **performance regression** in bash completion for large repositories (1–2 s hang for `git add ./<tab>` in repositories with ~368,000 files). The root cause is `git ls-files --others --modified --directory` with a wildcard pattern, forcing Git to scan every directory. Hughes proposed a workaround: an environment variable `GIT_COMPLETION_NO_COMPLETE_INDEX` to disable index-based completion.
- **[2026/08/14/19-38-49]** Colin Hinton’s patch removing the unused `name` parameter from the `chdir_notify` API was updated to document the historical context of the parameter’s deprecation, addressing Jeff King’s review feedback. The patch is ready for merging.