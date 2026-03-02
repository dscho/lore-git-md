# Git Mailing List Digest — 2025/03/05

## The day in brief

A moderately busy day with 41 emails across 15 threads, dominated by technical refinements to reference transactions and cruft pack handling. The standout developments include Karthik Nayak's v3 series implementing partial reference transactions and Taylor Blau's resolution of cruft pack freshening issues. Meanwhile, Git for Windows 2.49.0-rc1 brings notable deprecations.

## Notable threads

### Partial reference transactions reach v3

Karthik Nayak's 8-patch series introducing partial reference transaction support has reached its third iteration ([1](2025/03/05/17-38-55), [2](2025/03/05/17-38-59), [3](2025/03/05/17-39-00), [4](2025/03/05/17-39-01), [5](2025/03/05/17-39-02), [6](2025/03/05/17-39-03)). The implementation now includes a new `--allow-partial` flag for `git update-ref` that enables individual reference updates to fail while others proceed. Key refinements in this version include:

- A dedicated `ref_transaction_rejections` struct for efficient error tracking
- Consolidated error handling via `ref_transaction_maybe_set_rejected()`
- Support for partial failures during F/D conflict checks
- Comprehensive test coverage across 8 failure scenarios

Junio Hamano later raised philosophical questions about the "partial transaction" terminology ([2025/03/05/19-28-39]), suggesting "non-transactional batched updates" might better describe the behavior. The technical implementation appears settled after extensive review rounds with Patrick Steinhardt and Phillip Wood.

### Cruft pack freshening fix finalized

Taylor Blau's series addressing object freshening in multi-cruft pack scenarios has been reduced to a single focused fix ([2025/03/05/00-15-14], [2025/03/05/00-15-18]) after withdrawing an earlier simplification that risked creating perpetual repacking cycles. The final patch modifies `want_found_object()` in `pack-objects.c` to:

- Properly handle mtime comparisons for objects in retained cruft packs
- Introduce `ignore_packed_keep_in_core_has_cruft` flag
- Add `want_cruft_object_mtime()` helper
- Maintain existing behavior for non-cruft packs

The thread reached consensus after addressing concerns from Patrick Steinhardt about pathological repacking scenarios when `--max-cruft-size` approaches `--max-pack-size`.

### Git for Windows 2.49.0-rc1 announced

Johannes Schindelin announced Git for Windows 2.49.0-rc1 ([2025/03/05/14-52-33]) with notable changes including:

- Planned phase-out of `git svn` support
- Discontinuation of 32-bit installers (except MinGit until 2029)
- Updated dependencies (OpenSSH 9.9.P2, PCRE2 10.45)
- Fixes for Windows-specific issues including symlink renaming regressions and VS Code terminal hangs

The release incorporates upstream Git v2.49.0-rc1 changes including the `--name-hash-version=2` stabilization and `git backfill` command.

## In brief

**Reference backend consistency**: Taylor Blau and Jeff King discussed fixing inconsistent `transfer.hideRefs` behavior between packed and loose ref backends ([2025/03/05/23-12-54]), with Junio noting the recurring theme of moving common functionality to the generic refs layer ([2025/03/05/23-45-05]).

**Bash completion fixes**: David Mandelberg's v2 series improved remote name completion for slashed remotes ([2025/03/05/00-07-27]), introducing a path component counting helper and optimizing `for-each-ref` calls ([2025/03/05/20-50-20]).

**Build system docs**: Phillip Wood finalized conditional documentation for deprecated features using `WITH_BREAKING_CHANGES` ([2025/03/05/10-42-37]), while Junio fixed an `EXCLUDED_PROGRAMS` inconsistency for `git pack-redundant` ([2025/03/05/15-53-29]).

**Mentoring program docs**: The thread about subject prefixes for mentoring programs concluded with documentation updates to use "[GSoC PATCH]" format ([2025/03/05/02-06-26], [2025/03/05/14-26-32]).

**Language-specific gitignore**: Proposals for built-in language ignore templates met resistance from maintainers ([2025/03/05/22-43-26], [2025/03/05/22-52-49]), with Junio emphasizing Git's neutrality and suggesting the existing `--template` mechanism.

**GSOC project inquiry**: Ayush Chandekar explored refactoring `core.attributesfile` access as part of the `the_repository` removal effort ([2025/03/05/10-46-49]), receiving guidance from Jialuo about focusing on simpler global states first ([2025/03/05/11-41-41]).

## On the radar

The `transfer.hideRefs` inconsistency between backends appears headed for a solution at the `refs_ref_iterator_begin()` level, which would unify behavior for both packed and reftable backends. This aligns with the project's broader trend of moving common functionality from backends to the generic refs layer.