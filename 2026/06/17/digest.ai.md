Here’s the digest for **2026/06/17**, structured to highlight the day’s most consequential developments while keeping it concise and actionable for a developer loosely following Git’s evolution.

---

### **The day in brief**
A **heavy-traffic day** (97 emails, 26 threads) dominated by **ODB abstraction work**, **ref backend consolidation**, and **release stabilization** for Git 2.55.0-rc1. The most critical threads:
- **Patrick Steinhardt’s ODB abstraction series (`ps/odb-source-packed`)** reached v3, with Junio acknowledging it as mechanically sound but blocked by unresolved refdb lifecycle issues.
- **`git refs` subcommands (`ps/refs-writing-subcommands`)** were **merged**, consolidating reference manipulation under a unified interface.
- **Git 2.55.0-rc1** was announced, marking the start of the stabilization phase.
- **Controversial threads** (e.g., `git rebase --squash`, `git branch --delete-merged`) saw design debates but no resolutions.

---

### **Notable threads**

#### **ODB abstraction: Packed source conversion (`ps/odb-source-packed`)**
**What happened**: Patrick Steinhardt posted v3 of his 17-patch series converting `struct packfile_store` to `struct odb_source_packed`, completing the ODB abstraction work. Junio Hamano acknowledged the series as mechanically sound but noted it remains blocked by unresolved refdb lifecycle issues (memory leaks, duplicate refdb creation) requiring an 11-patch fix series.
**Why it matters**: This is the **final large-scale ODB restructuring** Patrick plans, with future work expected to be incremental. The series touches core object storage, MIDX interfaces, and repository setup, making it foundational for future backend work (e.g., reftable, geometric repacking).
**State**: **Blocked** (awaiting refdb fixes). Junio’s approval signals confidence in the design, but the series won’t progress until dependencies are resolved.
**Key detail**: The packed source now uses an embedded `struct odb_source` base, aligns with loose source interfaces, and registers all 13 required callbacks. Justin Tobler’s review confirmed type-system consistency but did not engage with deeper architectural concerns.

---

#### **`git refs` subcommands merged (`ps/refs-writing-subcommands`)**
**What happened**: Junio merged Patrick Steinhardt’s 5-patch series adding `git refs create|delete|update|rename` subcommands, consolidating reference manipulation functionality previously scattered across `git-update-ref`, `git-symbolic-ref`, and other commands.
**Why it matters**: This **unifies the CLI for reference operations**, improving discoverability and reducing fragmentation. The design resolves a key debate: `git refs update` retains `git update-ref`’s implicit creation semantics (for backwards compatibility), while `git refs create` provides an explicit atomic alternative.
**State**: **Merged to `master`**. The series is technically complete, with thorough test coverage and documentation.
**Key detail**: The `rename` subcommand defers reflog handling (no automatic reflog creation for new references if the source lacks one), leaving room for future refinement.

---

#### **Git 2.55.0-rc1 released**
**What happened**: Junio announced Git 2.55.0-rc1, incorporating 460 non-merge commits from 82 contributors. The release includes:
- **New builtin**: `git format-rev` (on-demand pretty formatting of revision expressions).
- **Hook parallelism**: Configurable concurrency for hooks (`hook.jobs`, `hook.<event>.jobs`).
- **Performance**: Optimized reachability bitmap generation, priority queues for revision traversal.
- **ODB abstraction**: Loose object source refactored into `struct odb_source`.
- **Windows/Linux fsmonitor**: Cross-platform daemon support.
**Why it matters**: This is the **first stabilization release** for Git 2.55, marking the shift from feature development to regression fixes. The sheer volume of changes (460 commits) suggests a significant release.
**State**: **Stabilization phase open**. No new features will be merged; focus is on testing and regression fixes.
**Key detail**: The release notes draft is comprehensive, covering CLI changes, performance improvements, and internal refactoring. Notable exclusions: Rust support is still opt-out, and the xdiff codebase is being prepared for Rust integration.

---

#### **`git rebase --squash` design debate**
**What happened**: Harald Nordgren’s proposal to add `--squash` to `git rebase` (to fold commit ranges while preserving messages) faced **architectural and efficiency concerns** from Phillip Wood. The debate centers on:
1. **Efficiency**: The current implementation picks commits one-by-one, risking repeated conflict resolution if the rebase base changes mid-operation. Phillip proposed a single three-way merge of the entire range (aligning with `git merge --squash`).
2. **Workflow integration**: Harald argues for tight rebase integration (automatic range inference from upstream), while D. Ben Knoble and Patrick Steinhardt propose `git history squash` as a more flexible alternative.
**Why it matters**: This touches on **Git’s history-editing philosophy**: whether to prioritize workflow integration (rebase) or architectural fit (standalone tools like `git history`).
**State**: **Stalled**. No resolution on architectural direction; Harald has not yet addressed Phillip’s efficiency critique.
**Key detail**: The thread highlights a tension between **discoverability** (rebase’s automatic range inference) and **flexibility** (`git history`’s rev-list/pattern-based range selection).

---

#### **`git branch --delete-merged` UX debates**
**What happened**: Harald Nordgren’s 15-iteration series adding `git branch --delete-merged` (safe automated branch cleanup) is **blocked on API consistency**. Phillip Wood flagged two mechanical issues:
1. `delete_branches()` should unpack all flags (including `force`) at the start for future-proofing.
2. `delete_merged_branches()` still uses a boolean `dry_run` parameter instead of the flags-based API.
**Why it matters**: This is a **high-utility feature** for workflows involving frequent branch cleanup, but the API design affects extensibility.
**State**: **Blocked** (awaiting API cleanup). The series is otherwise technically complete, with comprehensive test coverage and safety mechanisms (e.g., per-branch opt-out via config).
**Key detail**: The `--dry-run` test integration is also blocked until the API consistency issues are resolved.

---

### **In brief**
- **`git cat-file --batch-command` remote object info**: Pablo Sabater’s security-hardened series (v12) saw **surface-level review** of capability validation loop mechanics. The series is **technically settled** and ready for merging, with no outstanding objections.
  > **Remote object metadata** -- Pablo Sabater’s `git cat-file --batch-command` series (v12) resolves dynamic capability-based validation for querying object metadata from remotes without full downloads. The series is ready for merging after 12 iterations.

- **Windows LLP64 large-object support**: Philip Oakley’s series (merged to `next`) fixes 4GB+ object handling on Windows by converting `unsigned long` to `size_t`. Junio adopted GitGitGadget’s `--creation-factor=95` for local range-diff workflows.
  > **Large-object fixes** -- Windows LLP64 support for objects >4GB (merged to `next`) converts size-related variables from `unsigned long` to `size_t`, fixing truncation issues.

- **`fetch.followRemoteHEAD` config**: Matt Hunter’s 7-patch series (v2) adding a global `fetch.followRemoteHEAD` setting was **approved**, with Junio noting a minor design shortcoming in configuration value validation (silent ignoring of invalid values).
  > **Fetch config** -- Global `fetch.followRemoteHEAD` setting (merged) acts as a fallback for per-remote configurations, following the `fetch.prune`/`remote.*.prune` pattern.

- **Config include disabling retracted**: Derrick Stolee **retracted** his proposal to add `GIT_CONFIG_INCLUDES=0` and `--no-includes` after substantive concerns about security trade-offs and workflow breakage. The thread underscores Git’s preference for **architectural rigor** over piecemeal fixes.
  > **Config includes** -- Proposal to disable config includes via `GIT_CONFIG_INCLUDES=0` or `--no-includes` was **retracted** due to security concerns and workflow breakage risks.

- **`git rebase --abort` hints**: Harald Nordgren’s patch adding `--abort` hints to rebase failure messages reached **consensus** on design (split messages for exec failures, neutral tone). The series is ready for v2.
  > **Rebase UX** -- `--abort` hints added to rebase failure messages (consensus reached) to improve discoverability without implying workflow recommendations.

- **Zsh completion for `git -C`**: Lutz Lengemann’s patch fixing completion for `git -C <path> <command>` saw **substantive review** from Junio, who noted the fix should extend to other global options (`--git-dir`, `--work-tree`, `-c`).
  > **Zsh completion** -- Fix for `git -C <path> <command>` completion (v1) addresses `-C` but leaves other global options (`--git-dir`, `--work-tree`) broken.

- **`ignore_case` libification**: Justin Tobler and Tian Yuchen’s 2-patch series moving `ignore_case` into `struct repo_config_values` faced **substantive review** from Junio, who questioned the getter’s robustness during early initialization.
  > **`the_repository` removal** -- `ignore_case` libification (v1) moves the global variable into `struct repo_config_values` but faces questions about getter design for early initialization.

---

### **On the radar**
- **`git history drop`**: Patrick Steinhardt’s new `git history drop` subcommand (part of the `git history` roadmap) was introduced in `What’s cooking`. This is a **long-term effort** to provide more flexible history editing tools.
- **`mm/diff-process-hunks`**: Michał Kępień’s `diff.<driver>.process` feature (external hunk providers) is **stalled** due to security/performance concerns. This could enable powerful integrations with external diff tools.
- **`mv/log-follow-mergy`**: Martin von Zweigbergk’s `git log --follow` for non-linear history is **stalled** and needs attention. This would improve merge-commit tracking in `git log --follow`.
- **ODB abstraction follow-ups**: Patrick Steinhardt’s `ps/refs-avoid-chdir-notify-reparent` (ref backend chdir avoidance) and `ps/setup-drop-global-state` (setup.c refactoring) are **awaiting review**. These are part of the broader ODB abstraction effort.

---

### **Editorial note**
Today’s traffic reflects Git’s **dual focus** on **stabilization** (2.55.0-rc1) and **long-term refactoring** (ODB abstraction, `the_repository` removal). The **most critical action items** for contributors:
1. **Test Git 2.55.0-rc1** and report regressions.
2. **Review stalled topics** (e.g., `mm/diff-process-hunks`, `mv/log-follow-mergy`).
3. **Engage with design debates** (e.g., `git rebase --squash`, `git history` vs. rebase integration).

The ODB abstraction work (`ps/odb-source-packed`) remains the **most consequential long-term effort**, but its progress is gated by refdb fixes. Expect this to dominate discussion in the coming weeks.