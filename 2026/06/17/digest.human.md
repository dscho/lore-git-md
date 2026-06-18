# The day in brief

**2026-06-17** was a busy day on the Git mailing list, with **97 emails across 26 threads**. The tone was productive but with several significant design discussions still unresolved. **Notable developments**: Git v2.55.0-rc1 was released, several long-running series reached key milestones, and multiple architectural debates about ODB abstraction, reference management, and history editing continued. The day also saw the formal retraction of a controversial config-include-disabling feature.

---

## Notable threads

### Git v2.55.0-rc1 released
**What happened**: Junio C Hamano announced the first release candidate for Git 2.55, incorporating 460 non-merge commits from 82 contributors. The release includes new features like parallel hook execution, the `git format-rev` builtin, and fsmonitor support for Linux, along with significant performance optimizations and internal refactoring.

**Why it matters**: This marks the start of the stabilization period for Git 2.55. The sheer volume of changes—including ongoing efforts like ODB abstraction, `the_repository` removal, and Rust integration—makes this a significant release. The -rc1 tarballs are now available for testing, and the focus shifts to regression fixes and documentation updates.

---

### ODB abstraction reaches mechanical completion
**Topic**: [PATCH v3 0/17] odb: convert packed object source to `struct odb_source`
**Author**: Patrick Steinhardt

**What happened**: Patrick posted the third iteration of his 17-patch series converting the packed object storage backend into a proper `struct odb_source`. The series is now mechanically complete, with all callbacks wired up and MIDX interfaces refactored to use the concrete packed source type. Junio and Justin Tobler provided surface-level reviews confirming the type-system alignment, but the series remains blocked by unresolved refdb lifecycle issues (memory leaks, duplicate refdb creation) that Patrick is addressing in a separate 11-patch fix series.

**Why it matters**: This is the final large-scale ODB restructuring Patrick plans to undertake. The series represents a major step toward pluggable object storage backends, but its progress is now tied to the refdb fixes and an earlier dependency (`ps/setup-centralize-odb-creation`). The mechanical work is done; the remaining challenges are architectural.

---

### `git refs` subcommands merged
**Topic**: [PATCH v2 0/5] refs: add delete, update, create, and rename subcommands to `git refs`
**Author**: Patrick Steinhardt

**What happened**: Junio queued Patrick’s five-patch series adding reference-writing subcommands (`create`, `delete`, `update`, `rename`) to `git refs`. The series consolidates functionality previously scattered across `git-update-ref` and `git-symbolic-ref`, improving discoverability and reducing fragmentation. The design debate about atomicity vs. backwards compatibility was resolved in favor of retaining `update`’s implicit creation behavior (for script compatibility) while adding a dedicated `create` subcommand for explicit atomic creation.

**Why it matters**: This is a user-facing improvement that simplifies reference manipulation, particularly for scripting and automation. The series also aligns with Patrick’s broader ref backend modernization efforts, including the ongoing ODB abstraction work.

---

### Config-include-disabling feature retracted
**Topic**: [PATCH 0/3] Add ways to disable config includes
**Author**: Derrick Stolee

**What happened**: Derrick formally retracted his three-patch series proposing mechanisms to disable Git config include directives (`GIT_CONFIG_INCLUDES=0` and `--no-includes`). The retraction followed substantive concerns from Jeff King (Peff) about security trade-offs (partial coverage leaving other config vectors open) and workflow breakage (disabling includes could break legitimate use cases like conditional includes for work/personal email separation). The discussion shifted toward a broader sandboxing mechanism, but no revised proposal has been submitted.

**Why it matters**: The retraction underscores the project’s preference for architectural rigor over piecemeal fixes. The unresolved sandboxing problem (third-party tools failing due to config includes) remains a potential area for future work, but the binary disable approach is definitively off the table.

---

### `git rebase --squash` stalls on architectural concerns
**Topic**: [PATCH v2] rebase: add `--squash` option
**Author**: Harald Nordgren

**What happened**: Harald’s patch adding a `--squash` option to `git rebase` (to fold commit ranges while preserving original messages) faced decisive feedback from Phillip Wood. The primary concern is efficiency: the current implementation picks commits one-by-one, risking repeated conflict resolution if the rebase base changes mid-operation. Phillip proposed a more efficient approach (computing a three-way merge of the entire range against upstream in a single step, similar to `git merge --squash`). D. Ben Knoble and Patrick Steinhardt also suggested `git history squash` as a more flexible alternative, aligning with Patrick’s `git history` roadmap.

**Why it matters**: The thread highlights a tension between workflow integration (Harald’s preference for tight rebase coupling) and architectural fit (the `git history` alternative). The efficiency critique is a blocker for the rebase implementation, and the discussion remains unresolved. Expect a v3 iteration or a pivot to `git history`.

---

### `git branch --delete-merged` nears completion
**Topic**: [PATCH v15] branch: add `--delete-merged` option
**Author**: Harald Nordgren

**What happened**: Harald’s 15-iteration series adding `--delete-merged` to `git branch` (for safe automated cleanup of merged local branches) is now blocked on two API consistency issues. Phillip Wood’s feedback identified mechanical problems in the flags-based API, specifically:
1. The `force` flag should be unpacked at the start of `delete_branches()` for future-proofing.
2. The boolean `dry_run` parameter in `delete_merged_branches()` should be replaced with the flags-based API.

Harald acknowledged both issues and is preparing a final mechanical patch to align the API.

**Why it matters**: This is a mature, well-reviewed series that addresses a common workflow pain point. The remaining issues are minor and uncontroversial, so expect this to merge soon after the API cleanup.

---

## In brief

> **`git cat-file --batch-command` remote object info** -- Pablo Sabater’s security-hardened series for querying object metadata from remotes without full downloads is now technically settled. Chandra Pratap’s final review raised mechanical concerns about the capability validation loop, but the series remains ready for integration.

> **`git-interpret-trailers` documentation** -- Kristoffer Haugsbakk’s 11-patch documentation series clarifying trailer parsing rules and conventions was merged. The series resolves prior ambiguity about valid key formats, trailer ordering, and the `Based-on-patch-by` trailer’s legal requirements.

> **`git log --graph` indentation** -- Pablo Sabater’s v5 series improving the rendering of commits with excluded parents faces a critical interaction with the prio-queue refactoring in `revision.c`. Jeff King (Peff) identified that the lookahead logic in `graph_peek_next_visible()` misses commits already in the prio-queue, potentially leading to incorrect graph rendering. The series is blocked until this is resolved.

> **`git checkout`/`git switch --track=fetch`** -- Harald Nordgren’s v13 series adding `--track=fetch` to automatically fetch from the remote when creating a tracking branch was rejected by Junio. The maintainer remains unconvinced about the workflow value of allowing checkout to fetch from the network without user preview, despite the series’ technical completeness.

> **Windows LLP64 large-object support** -- Johannes Schindelin’s series converting object header lengths from `unsigned long` to `size_t` to handle objects >4GB on Windows was merged. The discussion also clarified GitGitGadget’s range-diff mechanics (`--creation-factor=95`), which Junio plans to adopt in his local workflow.

> **Global `fetch.followRemoteHEAD`** -- Matt Hunter’s series adding a global `fetch.followRemoteHEAD` configuration (falling back to per-remote settings) is ready to merge. Junio identified a design shortcoming in configuration value validation (silently ignoring invalid values), but this is not a blocker.

> **`git switch`/`git checkout --create-if-missing`** -- A new patch series from an unnamed author adds `--create-if-missing` to both commands for idempotent branch switching. The implementation is uncontroversial, but the maintainer’s earlier feedback about use-case validity remains unresolved.

> **Ref backend refactoring** -- Patrick Steinhardt’s eight-patch series modernizing Git’s reference backend infrastructure received substantive review from Justin Tobler. The most significant feedback targeted patch 7/8, where Justin questioned the design choice to repurpose `ref_storage_format` as an initialization state signal. The series is otherwise ready for integration.

> **SubmittingPatches trailer conventions** -- Kristoffer Haugsbakk’s five-patch documentation series updating `SubmittingPatches` for trailer usage conventions was merged. The series standardizes guidance on crediting contributions, avoiding Linux-specific trailers, and trailer ordering.

> **Patch review etiquette** -- Weijie Yuan’s RFC series proposing updates to `MyFirstContribution.adoc` and `SubmittingPatches` about patch review etiquette (requiring discussion before rerolling, reroll cadence guidelines) is nearing consensus. Junio’s feedback on the cadence guidance (e.g., whether to allow faster rerolls for "nearly accepted" topics) remains unresolved.

> **`git rebase --abort` hints** -- Harald Nordgren’s patch adding `--abort` hints to rebase failure and status messages reached consensus. The exec-failure message will now split into two parts (failure explanation + neutral `--abort` hint), preserving the existing warning while adding the escape hatch.

> **CI infrastructure for Windows builds** -- Patrick Steinhardt’s patch replacing Chocolatey with a PowerShell script for dependency management in GitLab CI was reviewed by Justin Tobler. The change reduces setup time from ~3 minutes to ~1 minute and improves reliability. Justin raised maintainability questions about version pinning frequency but did not challenge the design.

> **Sequencer notes handling** -- Uwe Kleine-König’s patch fixing notes copying from dropped commits during rebase was reviewed by Junio. The maintainer identified a fixup-commit edge case that could map notes to the wrong rewritten commit, blocking the patch until resolved.

> **macOS keychain credential helper** -- Toon Claes’s patch fixing a link error in the macOS keychain credential helper when Git is built with Rust support was merged. The fix aligns the build rule with other Git components by using `$(LIBS)` instead of `$(LIB_FILE) $(EXTLIBS)`.

> **Zsh completion for `git -C`** -- Lutz Lengemann’s patch fixing zsh completion for `git -C <path> <command>` was reviewed by Junio. The maintainer noted that the patch addresses `-C` but not other global options (`--git-dir`, `--work-tree`, `-c`), suggesting alignment with the bash implementation.

> **`ignore_case` libification** -- Justin Tobler and Tian Yuchen’s two-patch series moving the global `ignore_case` variable into `struct repo_config_values` received substantive review from Junio. The maintainer questioned the getter’s design (omitting the `repo->gitdir` check), asking whether all getters should follow the same pattern or if a more explicit initialization check is needed.

> **SubmittingPatches design justification** -- Junio posted a documentation patch requiring contributors to explicitly address high-level design critiques in resubmissions. The update aims to reduce patch series advancing through multiple iterations without resolving fundamental objections.

---

## On the radar

> **`git history drop`** -- Patrick Steinhardt’s new `git history drop` subcommand (to remove a commit and replay its descendants) was introduced in the "What’s cooking" report. This is part of an ongoing effort to provide more flexible history editing tools, but it lacks user feedback so far.

> **`git replay --linearize`** -- Thomas Koutcher’s `git replay --linearize` subcommand (to linearize a commit history) was also introduced in the "What’s cooking" report. Like `git history drop`, this is experimental and needs broader review.

> **`diff.<driver>.process`** -- Michał Kępień’s `mm/diff-process-hunks` topic (allowing external tools to act as hunk providers) is stalled due to security and performance concerns. This is a potentially powerful but complex feature that may generate discussion.

> **`git log --follow` for non-linear history** -- Martin von Zweigbergk’s `mv/log-follow-mergy` topic (extending `git log --follow` to handle non-linear history) is stalled and needs attention. This addresses a long-standing limitation in Git’s history traversal.

> **Configurable lock timeout** -- Jonathan Tan’s `jt/config-lock-timeout` topic (adding a configurable timeout for config file locking) is stalled. This is a practical improvement for high-contention environments.

> **`git branch --prune-merged`** -- Harald Nordgren’s `hn/branch-prune-merged` topic (adding `--prune-merged` to `git branch`) is controversial due to UX debates. The series is mature but needs a final push to resolve the remaining concerns.