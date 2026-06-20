# The day in brief

**2026/06/19** was a busy day on the Git mailing list, with **102 emails across 28 threads**. The day was dominated by **feature finalization** (notably `git cat-file --batch-command` for remote object metadata and `git history squash`), **refactoring efforts** (ODB abstraction, `the_repository` removal), and **design discussions** (pinned references, interactive typo correction). Several long-running series reached key milestones, while new bug reports and CI issues surfaced. The tone was **productive but detail-oriented**, with maintainers and contributors focusing on edge cases, test hygiene, and architectural consistency.

The **two threads you absolutely should not miss**:
1. **`git cat-file --batch-command` for remote object metadata** (Pablo Sabater) – the 13-iteration security-hardened series is now **fully ready for merging**, with all technical and security concerns resolved.
2. **`git history squash` ref-handling design** (Harald Nordgren) – the **final design decision** (reject by default, advise `--update-refs`) was reached, clearing the last blocker for this new command.

---

## Notable threads

### Git cat-file --batch-command for remote object metadata
**What happened**: Pablo Sabater’s 13-iteration series implementing `git cat-file --batch-command` support for querying remote object metadata (e.g., object sizes) without full downloads is now **fully ready for merging**. The v13 patch addresses the last mechanical concerns (dynamic capability-based validation, memory leak fixes, strict protocol enforcement) and includes comprehensive test coverage. Junio C Hamano and other maintainers have reviewed and approved the approach.

**Why it matters**: This feature enables efficient metadata queries over protocol v2, reducing network overhead for tools that need object sizes but not the full object data. The security hardening (input validation, capability-based allow-lists) makes it safe for production use.

**Current status**: **Merge-ready**. All substantive reviews are complete, and the series has been rebased on current `master`. The only remaining step is Junio’s final merge.

---

### Git history squash ref-handling design
**What happened**: The design for handling refs pointing to commits inside the squashed range in `git history squash` has been **finalized**. After discussion between Patrick Steinhardt, Junio C Hamano, and Harald Nordgren, the agreed behavior is to **reject the operation by default** if any refs point to commits inside the squashed range, with an `advice()` message suggesting `--update-refs[=head]` to explicitly retarget them. This conservative approach avoids surprising users by silently orphaning refs (e.g., bisection markers).

**Why it matters**: This resolves the last open design question for `git history squash`, a new command that collapses a range of commits into a single commit while preserving descendant history. The decision ensures the feature is safe for users who rely on refs for workflows like bisection or topic branches.

**Current status**: **Design finalized**; code changes pending to implement the ref-handling logic. The series remains in `next` until the implementation is updated.

---

### ODB abstraction: packed object source API
**What happened**: Patrick Steinhardt’s 17-part ODB abstraction series (v3) continues to progress, with Junio C Hamano resolving a minor type conflict in patch 15/17 via an evil merge. The series completes the conversion of packed object storage to `struct odb_source_packed` and exposes deeper refdb architectural flaws (memory leaks, duplicate refdb creation, initialization ordering). Junio has acknowledged the series as **mechanically sound and ready for integration once the blocking refdb issues are resolved**.

**Why it matters**: This is the **final large-scale ODB restructuring** Patrick plans to undertake, representing a major milestone in Git’s libification effort. The series establishes a clean separation between packed and loose ODB sources and aligns their interfaces.

**Current status**: **Blocked by refdb fixes**. Patrick is preparing an 11-patch fix series to address the refdb lifecycle issues, which depends on the "ps/setup-centralize-odb-creation" series being merged first.

---

### Ref backend refactoring: recursion prevention
**What happened**: Patrick Steinhardt’s 10-part refactoring series (v4) introduces a targeted `ignore_refs` flag in `struct config_options` to prevent recursive initialization during ref store setup. The series resolves the root cause of "onbranch" config conditions triggering ref store access while preserving other include conditions. Junio C Hamano and Jeff King have reviewed the approach, which directly addresses earlier architectural concerns.

**Why it matters**: This is a **critical fix for ref backend initialization**, which has been plagued by recursive issues. The `ignore_refs` flag provides a clean, low-risk solution that preserves other config include conditions (e.g., `includeif.gitdir`) while avoiding the circular dependency.

**Current status**: **Ready for review**. The series is well-motivated and aligns with project conventions. Patches 7–9/10 (the core recursion fix) may warrant closer scrutiny for edge cases in config parsing.

---

### MIDX incremental base regression fix
**What happened**: Taylor Blau’s 3-patch series fixes a regression in `git multi-pack-index write --incremental --base`, which ignored the custom base and broke reachability closure for bitmaps. The fix is **low-risk and well-scoped**, with a test helper (`nth_line()`) added for new test coverage. Junio’s "What’s cooking" report marks it as "Needs review."

**Why it matters**: The regression renders incremental repacking unusable when a custom base is specified, affecting users of MIDX and geometric repacking. The fix is critical for the usability of these features.

**Current status**: **Needs review**. Jeff King (Peff) has been CC’d, and the series is expected to graduate to `next` soon.

---

## In brief

> **`git branch --delete-merged` stacked-branch safety** -- Junio C Hamano and Phillip Wood reached consensus on the **abort-and-notify** default for stacked-branch protection, clearing the last design blocker for Harald Nordgren’s series. The feature will now preserve branches used as upstreams for unmerged stacked branches (e.g., `feature1` if `feature2` stacks on it) to avoid breaking `@{u}` references and implicit `git rebase` workflows.

> **`git rebase --abort` hints** -- Junio C Hamano questioned whether the `--abort` hint should be shown for *all* exec failures or only irrecoverable ones (e.g., broken scripts). Harald Nordgren’s patch adds the hint to exec-failure messages, but the discussion remains open on whether to conditionally suppress it for transient failures (e.g., missing files).

> **`git log <file>` subtree merge clarification** -- Junio C Hamano clarified that `git log <file>`’s omission of commits in subtree merges is **intentional**, not a bug. The workaround is to specify all historical paths explicitly (e.g., `git log -- path1 path2`). The `--reverse` behavior discrepancy remains unresolved.

> **Sparse-checkout data loss bug** -- A new bug report from charmocc reveals that `git checkout` in sparse-checkout mode **silently overwrites untracked files** and emits a misleading warning, unlike its behavior in normal mode. This inconsistency creates a risk of data loss and warrants prompt attention.

> **macOS CI timeout diagnosis** -- Patrick Steinhardt identified orphaned `httpd` and `git-remote-http` processes as the root cause of macOS CI job timeouts in GitHub Actions. The issue is specific to the `t5551` and `t5559` test scripts (HTTPD-based tests) and appears to be triggered only in GitHub’s macOS runner environment.

> **`ignore_case` libification** -- Justin Tobler and Tian Yuchen’s 2-patch series (v3) moves the global `ignore_case` variable into `struct repo_config_values` as part of the `the_repository` removal effort. The series is **merge-ready**, with all maintainer feedback incorporated.

> **`protect_hfs`/`protect_ntfs` libification follow-ups** -- Junio C Hamano proposed two follow-up patches for Tian Yuchen’s series: (1) replacing `repo->gitdir` with `repo->initialized` in accessor functions for consistency, and (2) consolidating accessor function comments for readability. Both are stylistic and deferred until after Git 2.55 final.

> **Interactive typo correction** -- The debate over Jishnu C K’s patch to enhance `help.autoCorrect=prompt` continues, with Junio C Hamano rejecting the idea of changing the default behavior. The discussion now centers on whether to **preserve arguments in the prompt** (e.g., `Run 'git checkout neo' instead [y/N]?`) as an incremental improvement to the existing config-based approach.

> **Pinned references RFC** -- Erik Östlund’s proposal for "pinned references" (a ref + expected OID invariant) remains in the exploratory phase. Junio C Hamano ruled out modifying the existing `<describeOutput>` syntax, leaving the discussion open on whether a new mechanism (e.g., a new command or option) is justified.

---

## On the radar

- **`git replay --linearize`** (Taylor Blau) – A new command to linearize a commit history while preserving merge commits. The series is in `next` but lacks substantive review.
- **`git refs create|delete|update|rename`** (Patrick Steinhardt) – A new plumbing command for ref management. The series is in `next` and awaits feedback on the user-facing design.
- **`mm/diff-process-hunks`** – A stalled series enabling external hunk providers via `diff.<driver>.process`. Junio’s "What’s cooking" report flags it as needing attention due to security/performance concerns.
- **`jt/config-lock-timeout`** – A stalled series adding config file locking retries. Junio’s report notes it needs review.
- **`mv/log-follow-mergy`** – A stalled series improving `git log --follow` for non-linear history. Junio’s report flags it as needing attention.