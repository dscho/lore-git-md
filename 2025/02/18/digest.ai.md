# Git Mailing List Digest — 2025/02/18

**The day in brief.** A busy day with 103 emails across 24 threads, featuring significant progress on multiple fronts. The standout developments include final approvals for the promisor-remote protocol capability and reftable library decoupling, along with active discussions about build system improvements and documentation fixes. Performance optimizations in the refs subsystem and a completed series for `git merge-tree --stdin` also merit attention.

## Notable threads

**Promisor-remote protocol capability finalized**  
Christian Couder's v5 patch series introducing promisor-remote protocol capability received Junio Hamano's approval after multiple iterations. The feature allows servers to advertise promisor remotes to clients during clone/fetch operations, simplifying large object handling in client-server scenarios. Key improvements in this version include case-sensitive URL comparison, removal of problematic foundational patches, and comprehensive test coverage. The accompanying Large Object Promisors (LOP) design document provides architectural context for these protocol changes.

**Reftable library decoupling complete**  
Patrick Steinhardt's 18-patch series to fully decouple the reftable library from Git core dependencies received final approval after addressing Windows build system conflicts. The changes enable standalone use by projects like libgit2 by systematically replacing Git-specific helpers with reftable-native implementations. The series culminates in a new compat/posix.h header providing the minimal POSIX layer needed, with all major technical concerns resolved through six iterations of review.

**Build system unification advances**  
Patrick Steinhardt also led a 12-patch series adding Meson build system support for Git's contrib components, including credential helpers, git-contacts, Coccinelle, and gitk. The work follows established patterns for Meson integration while handling platform-specific requirements. Notable aspects include centralized source directory handling, cross-platform CI verification, and the extraction of build logic into reusable scripts. The series leaves only git-gui, sparse, and perf tests without Meson support.

**Merge-tree stdin improvements finalized**  
Phillip Wood completed a 5-patch series improving `git merge-tree --stdin` reliability with deadlock resolution via stdout flushing between merges (<0.5% overhead). The changes include proper scoping of merge configuration to plumbing needs and thorough documentation updates clarifying stdin mode behavior. Elijah Newren provided final review approval, noting the careful balance between plumbing characteristics and porcelain considerations.

## In brief

The OS version advertisement feature series received final approval with only minor documentation nits remaining, switching to dash-separated format (e.g., "git/1.8.3.1-Linux"). Perl version requirements were standardized across build systems in a completed series from Peter Oliver. A documentation patch standardized terminology between `git-commit.txt` and `git-rebase.txt`, completing earlier work. The pickaxe functionality (-G/-S) gained protection against empty string arguments that previously caused crashes. Cross-compilation path handling in Meson builds saw active discussion about shell path detection. Shell script diff recognition improvements progressed but hit test failures needing resolution. A bugfix addressed NFSv4 compatibility issues with packfile permissions during server failovers. HTTP authentication regressions with netrc files were reported and analyzed. Gitk received encoding fixes for Japanese filenames during merge conflicts.

## On the radar

The `format.from` configuration discussion continues regarding its effects on `git show --format=email`, with Junio Hamano emphasizing historical design intent versus practical needs. The commit-slab refactoring for `show-branch` flags storage awaits resolution of technical feedback about bit array handling. Submodule merge behavior placement in documentation remains under discussion between `git-merge.adoc` and merge-strategies.txt. A heated exchange about documentation patch style requirements may require maintainer intervention to resolve constructively.