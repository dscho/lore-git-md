# Git Mailing List Digest - 10 May 2025

**The day in brief.** A moderately busy Saturday with 50 emails across 16 threads, featuring significant progress on several fronts. Key developments include final refinements to Bash function recognition in diffs, continued debate over Change-ID standardization, and a major proposal for stash behavior changes under Git 3.0's breaking changes flag. Documentation standardization work also saw multiple patches land.

## Notable threads

**Bash function recognition finalized** -- Moumita Dhar's GSoC contribution to improve shell script function detection in diffs reached its final review stage. Johannes Sixt provided meticulous feedback on test cases for both the core functionality and word-diff behavior. The series now only needs minor regex adjustments for comment handling before being ready for merge. This represents the culmination of several iterations that have progressively refined the Bash userdiff driver's accuracy.

**Change-ID debate deepens** -- The discussion around standardizing Change-IDs took a philosophical turn as D. Ben Knoble questioned whether the problem being solved was sufficiently well-defined. Martin von Zweigbergk responded with concrete use cases from Jujutsu's implementation, highlighting benefits for local commit message updates and navigation. The thread now balances between abstract design questions and practical tooling needs, with no clear consensus yet on whether this belongs in Git core.

**Stash behavior overhaul proposed** -- A comprehensive series proposes changing `git stash apply/pop` to default to restoring the index when Git 3.0's breaking changes flag is enabled. The 9-patch series includes implementation, documentation updates, and extensive test modifications to handle both old and new behaviors. Surprisingly, the new behavior also fixes several known submodule edge cases. This well-motivated change addresses long-standing user confusion but may spark debate about migration impact.

**Documentation standardization complete** -- Jean-Noël Avila and Junio C Hamano landed the final patches in their man page standardization effort, systematically applying consistent formatting to `git-daemon`, `git-var`, `git-write-tree`, and the `verify-*` commands. The changes focus on SYNOPSIS conversion and option backquoting, completing work started in September 2024. This represents the quiet conclusion of a months-long documentation improvement initiative.

**Sparse index optimizations validated** -- Elijah Newren gave final approval to Derrick Stolee's sparse index integration for `git add -p/-i` and `git apply`, confirming dramatic performance improvements (96.7% runtime reduction) while addressing test case accuracy concerns. The series demonstrates Git's continued progress on large repository performance, particularly for monorepo workflows using sparse checkouts.

## In brief

**Reftable memory leak fix** -- A focused patch plugs a memory leak in the reftable writer when padded writes fail, adding proper cleanup in the error path.

**Submodule path reuse protection** -- A bugfix prevents accidental `.gitmodules` overwrites when reusing paths for new submodules, requiring explicit `--force` or `--name` when conflicts are detected.

**gitk Tcl/Tk compatibility** -- A regression requiring Tk 8.5+ was fixed by reverting to legacy widgets, restoring compatibility with older systems while maintaining functionality.

**Merge-tree optimization** -- A new `--mergeability-only` flag allows Git forges to check branch mergeability without computing full merge results, avoiding unnecessary object writes.

**FQDN detection improvement** -- `git send-email` gains a new fallback method for Fully Qualified Domain Name detection using `hostname --fqdn` on Linux/macOS systems.

**Interactive diff context controls** -- A completed series now lets interactive patch commands (`add -p`, `commit -p` etc.) respect `diff.context` configs and accept command-line options to override them.

## On the radar

**contrib/ cleanup policy** -- Patrick Steinhardt's effort to establish formal maintenance criteria for `contrib/` components awaits final policy documentation after resolving questions about removal rationale wording.

**Sparse-checkout symlink handling** -- Gabriel Scherer's proposal for better sparse-checkout symlink handling met resistance from Elijah Newren, who argues dependency analysis should remain outside Git core. The discussion appears stalled unless new approaches emerge.