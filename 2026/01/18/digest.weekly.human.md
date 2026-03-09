# Git Mailing List Digest - 2026/01/12 -- 2026/01/18

**The week in brief.** A busy week with 608 emails across 154 threads, featuring several major developments in Git's core functionality. The most significant progress came in three areas: Adrian Ratiu's submodule path encoding and Patrick Steinhardt's `git-history` command both reached completion after extensive review, while Taylor Blau's MIDX compaction series addressed critical corruption handling. The week also saw substantial discussion around security defaults for ANSI escape sequences and ongoing work in the object database abstraction effort. With multiple large series now merge-ready, the project appears poised for significant infrastructure improvements in the upcoming release cycle.

## Key developments

### Submodule path encoding finalized

Adrian Ratiu's 11-patch series implementing runtime configuration for submodule gitdir paths via `extensions.submodulePathConfig` was approved for merging in its v9 iteration. The feature addresses filesystem conflicts through a four-stage resolution process (plain name → URL-encoded → numbered → hashed fallback) with comprehensive case-folding protection. Production validation at Google confirmed the solution replaces their downstream implementation. This concludes a multi-month effort to robustly handle submodule paths across all filesystems, with Junio Hamano noting the series had reached "diminishing returns" for further improvements after addressing all feedback from Patrick Steinhardt and others.

### git-history command approved

Patrick Steinhardt's 8-patch series introducing the experimental `git history` command with initial `reword` subcommand completed its tenth iteration and received final approval. The implementation builds on Git's replay infrastructure to provide fast, in-memory history editing capabilities inspired by Jujutsu. After extensive review from Elijah Newren (merge/rebase expert) and resolution of SZEDER Gábor's forward-compatibility concerns, the command is now merge-ready. It works with dirty working trees and updates all local branches containing rewritten commits by default, though currently skips hook execution. A late-week debate about the safety implications of rewriting multiple branches by default was resolved in favor of the current design, with Junio Hamano agreeing the approach provides better recovery options via reflog.

### MIDX compaction with bitmap support

Taylor Blau's 18-patch series implementing MIDX compaction functionality with reachability bitmap support reached final review stages. The new `git multi-pack-index compact` subcommand combines adjacent MIDX layers while preserving bitmap compatibility, with comprehensive test coverage and careful attention to backward compatibility with the new MIDX v2 format. The series also fixed an edge case where MIDX writes could skip updates for corrupt files, splitting this fix from the larger work for v2.53.0 inclusion. Junio Hamano and Patrick Steinhardt provided thorough review feedback, with only minor documentation tweaks remaining before merging.

### Object database abstraction advances

Patrick Steinhardt's major refactoring of Git's object database layer progressed significantly with a 14-patch series introducing a new iteration API. The work systematically replaces direct access to loose and packed object storage with a unified `odb_for_each_object()` interface, a key step toward making the object storage backend pluggable. The series underwent thorough review from Justin Tobler and others, with Junio Hamano indicating it's ready for integration. This represents foundational work in Patrick's multi-year effort to abstract Git's object storage layer, with measured performance improvements including a 9% disk size lookup speedup in the object info handling refactoring.

### ANSI escape sequence security policy

Johannes Schindelin's series to secure Git's sideband channel against terminal injection attacks (CVE-2024-32002, CVE-2024-52005) reached consensus on a middle path proposed by Jeff King: security controls will be opt-in rather than enabled by default. The technical approach (bitmask flags for controlling color, cursor movement, and erase sequences) remains unchanged, but the policy debate about defaults settled on permissive defaults with opt-in security controls via `sideband.allowControlCharacters`. This preserves backward compatibility while allowing security-conscious users to lock down their configuration, though philosophical differences about the right default policy remain unresolved.

## In brief

**Ref backend validation unified** -- Patrick Steinhardt's 17-patch series to centralize ref consistency checks across files and reftable backends concluded, moving fsck checks into the refs subsystem with shared validation functions.

**Tree parsing freed from the_repository** -- René Scharfe completed the tree-parsing portion of the multi-year `the_repository` removal effort, introducing repository-aware variants of tree parsing functions.

**Submodule ignore behavior standardized** -- Claus Schneider's series aligning `git add` behavior with other commands for submodules configured with `ignore=all` was finalized using `--force` rather than the originally proposed flag name.

**git status shows push tracking** -- Harald Nordgren's long-running series to enhance `git status` with push tracking branch divergence was approved for `next`, with ongoing discussion about making comparisons configurable via `status.compareBranches`.

**Hook execution changes reverted** -- Junio Hamano reverted the entire 'ar/run-command-hook' topic for the 2.53 release cycle due to regressions, planning to rework the changes as a new topic later.

**Lisp dialect support unified** -- Scott Burson's work to extend Git's diff highlighting to support Common Lisp and Emacs Lisp alongside Scheme reached completion under a single "lisp" driver.

**Shallow fetch depth calculation fixed** -- Samo Pogačnik's series correcting relative-depth fetching in shallow repositories is now merge-ready after documentation polish.

**UTF-8 diffstat display** -- Lorenzo Pegorari's GSoC patch fixed incorrect truncation of UTF-8 filenames in diffstat output with comprehensive test coverage.

**Test modernization** -- Multiple contributors replaced legacy test patterns (`! test -f` -> `test_path_is_missing`) across several test scripts.

**CI optimization** -- Johannes Schindelin skipped git-svn tests in leak-checking CI runs, cutting 42 minutes (13%) from job times.

## Looking ahead

**Rust integration** -- Ezekiel Newren's xdiff refactoring with Rust compatibility continues under review, with Phillip Wood and René Scharfe providing detailed feedback on the `ivec` implementation that's being addressed.

**Geometric repacking with promisor packs** -- Patrick Steinhardt's series wasn't included in 'seen' yet as Junio wants to review it further before integration.

**GSoC 2026 planning** -- Christian Couder initiated early discussion for Git's potential involvement, seeking mentors and administrators with applications opening January 19.

**Hook subsystem redesign** -- The discussion around stdin/stderr handling in hooks may lead to more fundamental design changes in future iterations following the temporary revert.

**Push negotiation optimizations** -- Threads exploring inefficient data transfer during branch creation may lead to protocol improvements in upcoming cycles.