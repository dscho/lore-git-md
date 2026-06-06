# Here's the daily digest for 2026/06/05:

## The day in brief

A moderately active Friday with 58 emails across 18 threads, featuring the completion of Harald Nordgren's long-running `git branch --delete-forked` series (now at v13), continued discussion on submodule mirroring strategies, and several performance optimizations nearing completion. The most notable development is the finalization of the safe branch deletion feature after 13 iterations of review.

## Notable threads

### Safe branch deletion reaches final form

Harald Nordgren's **`git branch --delete-forked`** series (formerly `--prune-merged`) has reached its final v13 iteration after extensive review. The feature provides safe automated cleanup of local branches whose work has been integrated upstream, with comprehensive safety mechanisms including configurable branch exemptions, dry-run previews, and warnings for unmerged branches. Phillip Wood's final reviews focused on implementation refinements like using `strvec` instead of `string_list` and consolidating boolean parameters into a flags bitfield. The series now cleanly integrates with Git's ref-filter machinery and maintains all safety guarantees while providing maximum user control.

### Submodule mirroring debate continues

The discussion about **submodule mirroring strategies** deepened with Simon Richter's argument that the core issue stems from conflating a submodule's identity with its location. The thread now features three technical approaches under consideration: repository-level mirroring (Junio's preferred solution), transitional submodule fallback URLs (Benson Muite's original approach), and Matt Hunter's suggestion to leverage Git's existing but underused remote multi-URL support. The debate highlights tensions between immediate archival needs (Debian's use case) and Git's architectural principles, with no clear consensus yet on the best path forward.

### Worktree metadata tracking proposed

Norbert Kiesel's proposal to add **worktree metadata tracking** gained stronger justification today with both practical examples from Kristoffer Haugsbakk (build environments, version snapshots) and philosophical grounding from Chris Torek arguing that worktrees are general-purpose workspaces distinct from branches. The discussion continues to debate whether creation timestamps or last-updated timestamps would be more useful, with Phillip Wood maintaining skepticism about the feature's necessity despite the strengthened rationale.

### Performance optimizations near completion

Two performance optimizations are nearing merge readiness:
- Arijit Banerjee's **delta base caching** for index-pack has addressed all technical concerns, maintaining 13-16% speedups while carefully managing memory usage. Reviewers are satisfied with either v2 or v3 of the series.
- Kristofer Karlsson's **priority queue optimizations** now have benchmark results showing both the cascade-down and lazy sift-down approaches provide comparable speedups (3-8% depending on workload. The thread has shifted from approach comparison to planning a phased integration strategy.

## In brief

**Ref-filter performance fix** -- Restores efficient ref iteration in `git branch` and `git tag` when many loose refs exist, fixing a regression from the `--start-after` feature. Benchmarks show improvements from ~2.8s to ~0.1s with 10,000 refs.

**Repository initialization refactoring** -- Patrick Steinhardt's series centralizing object database setup received Karthik Nayak's final approval, completing this phase of the ODB abstraction effort.

**Documentation standardization** -- Kristoffer Haugsbakk concluded his series synchronizing `replay.refAction` documentation between man pages with final formatting polish.

**Compiler compatibility** -- First-time contributor Dominik Loidolt's patch adding Clang version checking for UNUSED macro warnings was approved after tracing the GCC bit-shift style to glibc origins.

**Worktree improvements proposed** -- Jason Newton introduced copy-on-write worktree creation and shared-branch worktrees, though brian m. carlson raised concerns about dirty worktree states during reflink operations.

## On the radar

**Global remote.HEAD tracking** -- Matt Hunter proposed a global remote.followRemoteHEAD setting to avoid per-remote configuration, addressing a management pain point for consistent behavior across repositories.