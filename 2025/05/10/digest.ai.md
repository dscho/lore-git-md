# Git Mailing List Digest - 2025/05/10

**The day in brief.** A moderately busy Saturday with 50 emails across 16 threads, featuring several notable developments. The standout items include a proposed breaking change to `git stash` behavior, performance optimizations for merge operations, and continued refinement of the `contrib/` cleanup policy. Documentation standardization work also makes steady progress.

## Notable threads

### Stash behavior change proposed for Git 3.0

A comprehensive series proposes making `git stash apply/pop` default to restoring the index (equivalent to current `--index` behavior) when breaking changes are enabled. The change addresses long-standing user confusion where carefully staged changes are lost during stash operations. The implementation is cleanly gated behind the `WITH_BREAKING_CHANGES` flag and includes thorough test updates across all stash subcommands. Interestingly, the new behavior unexpectedly fixes several known submodule edge cases, providing additional justification for the change. Documentation updates in BreakingChanges.adoc clearly explain the motivation and new workflow.

### Merge-tree optimization for Git forges

A new `--mergeability-only` option is proposed for `git merge-tree` to optimize server-side merge checks. When enabled, the command exits early upon detecting any conflict without computing full merge results or writing objects to the ODB. The two-patch series first adds internal plumbing in merge-ort for early termination, then exposes the feature through the command-line interface. This targets Git forges like GitHub and GitLab that frequently need to check mergeability without requiring complete merge output. Tests verify both the early exit behavior and that no objects are written in this mode.

### Sparse index performance improvements finalized

Derrick Stolee's sparse index optimization series for `git add -p/-i` and `git apply` reaches completion after addressing Elijah Newren's test accuracy concerns. The final performance tests show dramatic improvements - a 96.7% runtime reduction (2.09s to 0.07s) for sparse index operations. The thread demonstrates Git's rigorous review culture, with multiple rounds of test refinement ensuring the optimizations handle edge cases correctly, particularly around out-of-cone file modifications and index state management between operations.

### Contrib cleanup policy refined

The `contrib/` directory cleanup discussion shifts from technical execution to formal policy definition. Junio Hamano weighs in against using "no updates in 5 years" as a standalone removal rationale, suggesting it could invite unnecessary pushback. The emerging consensus favors technical brokenness or clear alternatives as primary removal criteria, with time-based signals as secondary indicators. Practical concerns also surface regarding tools like `git-contacts` that, while unmaintained, still serve specific workflow needs with low friction.

### Change-ID semantics debated

The Change-ID standardization discussion deepens with Martin von Zweigbergk providing concrete examples from Jujutsu's implementation. The exchange highlights both local workflow benefits (commit message updates, navigation) and distributed system challenges (predecessor tracking, graph integrity). D. Ben Knoble's earlier call for clearer problem definition appears to shape the conversation toward more fundamental questions about what problems Change-IDs should solve in Git's object model.

## In brief

**Bash function pattern improvements** -- Moumita Dhar's GSoC contribution for better shell function recognition in diffs receives final polish from Johannes Sixt, addressing regex edge cases for commented function declarations.

**Documentation standardization** -- Jean-Noël Avila and Junio Hamano complete the mark-up updates for several commands (git-daemon, git-var, git-write-tree, git-verify-*), finalizing environment variable formatting and option documentation.

**FQDN detection enhancement** -- `git send-email` gains improved FQDN detection using `hostname --fqdn` as an additional fallback method on Linux/macOS systems.

**Submodule path reuse protection** -- A bugfix prevents accidental `.gitmodules` overwrites when reusing paths for new submodules, requiring explicit `--force` or `--name` when conflicts are detected.

**Reftable memory leak fix** -- Patrick Steinhardt plugs a memory leak in the reftable writer when padded writes fail, adding proper cleanup in the error path.

**Gitk Tcl/Tk compatibility** -- A regression requiring Tk 8.5+ is fixed by replacing `ttk::combobox` widgets with a backward-compatible alternative, restoring functionality on older systems.

## On the radar

**Sparse-checkout symlink handling** -- Gabriel Scherer's proposal for automatic handling of symlink targets in sparse checkouts meets resistance from Elijah Newren, who argues it would create more problems than it solves. The discussion highlights fundamental tensions in sparse-checkout's design philosophy.