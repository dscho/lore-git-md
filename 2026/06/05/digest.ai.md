Here's the daily digest for June 5, 2026:

## The day in brief

A moderately busy Friday with 58 emails across 18 threads, featuring the completion of several long-running series and substantive discussions about worktree improvements and submodule mirroring. The standout developments include Harald Nordgren's final v13 of `git branch --delete-forked` and Patrick Steinhardt's ODB refactoring series receiving final approval.

## Notable threads

### `git branch --delete-forked` reaches final form

Harald Nordgren's 13-iteration series adding safe automated branch cleanup is now complete with v13. The final version incorporates all review feedback, including:
- Reworked `--forked` filter integration with ref-filter machinery
- Replacement of boolean parameters with a flags bitfield
- Positional argument handling for branch patterns
- Comprehensive dry-run` support showing planned deletions

The series maintains all safety checks from `git branch -d` while adding configurable opt-outs, warn-only mode for unmerged branches, and now dry-run capability. With all technical concerns addressed through collaborative review, this appears ready for merging.

### Worktree improvements proposal faces technical scrutiny

Jason Newton proposed two worktree enhancements via GitGitGadget:
1. Copy-on-write worktree creation (`--reflink`) for supported filesystems
2. Allowing shared-branch worktrees (multiple checkouts of same branch)

brian m. carlson raised substantive concerns about edge cases:
- Potential stale files when reflinking dirty worktrees
- Race conditions with concurrent commits on shared branches
- Need for Windows ReFS support documentation

The proposal shows promise for workflows needing many worktrees, but requires resolution of these edge cases before proceeding.

### Submodule mirroring debate continues

The discussion about submodule mirroring alternatives evolved with new perspectives:
- Matt Hunter noted Git's remotes already support multiple URLs (currently unused)
- Simon Richter framed the core issue as conflating identity (what) with location (where)
- Benson Muite provided concrete examples of submodule URL fallback needs

The thread now balances three approaches:
1. Repository-wide URL rewriting
2. Submodule-specific fallback URLs
3. Remote multi-URL unification

Debian's archival requirements continue driving the discussion while maintainers push for general solutions over submodule-specific changes.

### ODB refactoring complete

Patrick Steinhardt's 16-part series converting packed object storage to use `struct odb_source` received its first review from Karthik Nayak, who approved the initial mechanical rename. This foundational work for the ODB abstraction effort appears on track after extensive prior review of related patches.

## In brief

**Ref-filter performance fix** -- Restores efficient ref iteration in `git branch` and `git tag` when many loose refs exist, fixing a regression from the `--start-after` feature. Benchmarks show ~28x speedup.

**Priority queue optimizations** -- Kristofer Karlsson's benchmarks show both cascade-down and lazy sift-down approaches provide comparable speedups (3-8%), with plans to integrate both in phases.

**Worktree metadata tracking** -- Chris Torek provided philosophical justification for the feature, arguing worktrees are general workspaces needing their own metadata distinct from branch descriptions.

**Compiler compatibility** -- Dominik Loidolt's patch adding Clang version checking for UNUSED macro warnings completed review and is ready for merging.

**Documentation standardization** -- Kristoffer Haugsbakk's series synchronizing `git replay` config documentation between man pages concluded with all feedback addressed.

## On the radar

**`remote.followRemoteHEAD` default** -- Matt Hunter proposed a global default for this per-remote setting to reduce configuration overhead, building on Git 2.48's introduction of the feature.