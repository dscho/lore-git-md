# Git mailing list daily digest for 2026/08/23

## The day in brief
Delilah Ashley Wu’s three-patch series fixing Git’s `--global` config listing inconsistency was resubmitted as v2, addressing all prior feedback. Michael Montalbo proposed a major pivot in the diff provider RFC, abandoning the "diff provider" abstraction in favor of a simpler "cousin" interface to `xdiff-interface.h`. A new `git organize` command was introduced to restructure Git’s source tree into subsystem-specific directories, sparking discussion about modularity and tool design.

## Notable threads

### Global config listing inconsistency fixed (v2 resubmission)
Delilah Ashley Wu resubmitted her three-patch series to fix Git’s inconsistent `--global` config listing behavior, where `git config list --global` only showed `$HOME/.gitconfig` despite Git reading both that and `$XDG_CONFIG_HOME/git/config`. The v2 series addresses all prior feedback by reordering patches to avoid temporary regressions, narrowing Windows path normalization scope, and squashing test-only changes.

The core change modifies `config_sequence()` in `config.c` to handle the global scope by reading both config files, using new flags (`ignore_system`, `ignore_repo`) and counters (`global_config_success_count`) to track reads and enforce error handling. The series also includes preparatory work: patch 1 ensures forward slashes in XDG config paths on Windows, and patch 2 adjusts error handling to prevent regressions when unreadable config files are encountered. Johannes Schindelin’s Reviewed-by tags from v1 remain applicable to the technical aspects.

The series is now technically complete and ready for maintainer integration. Chris Torek raised a follow-up question about the write-side counterpart: which file `git config --global --edit` currently edits and which it *should* edit, but this does not challenge the series’ implementation.

### Diff provider RFC pivots to "cousin" interface
Michael Montalbo proposed a major pivot in the diff provider RFC, abandoning the "diff provider" abstraction introduced in v7 in favor of a simpler "cousin" interface to `xdiff-interface.h`. The new approach models the interface after the existing `diff.<driver>.process` plumbing, letting users plug in external diff tools via pkt-line while preserving xdiff as the final arbiter for content rendering.

The pivot addresses Phillip Wood’s core feedback by dropping the "authority" question (external tools are now advisory, not authoritative), replacing invented terminology ("coordinates", "trim-stable pairs") with established Git terms, and ejecting the `diff-hunks` store to avoid cache-keying concerns. The goals remain the same: enabling external diff tools to integrate with Git’s features (word diff, `--color-moved`, function context) and allowing xdiff-interface callers to operate on OIDs alone when only hunk headers are needed.

Montalbo plans to re-roll the series based on v6, focusing on the pkt-line external process provider. The discussion now centers on whether the "cousin" interface is the right abstraction for integrating external diff tools without disrupting Git’s built-in features.

### Reftable backend optimizations advance
Karthik Nayak’s three-patch series optimizing the reftable backend saw progress on two key design questions. The series targets redundant stack reloads during reference transactions, which were causing excessive `fstat()` calls and a 25% performance bottleneck in bulk ref operations (per Jeff King’s benchmarks).

In response to Junio C Hamano’s safety concern about concurrent `reftable_addition` instances, Nayak proposed adding a tracking bit to `struct reftable_addition` to record whether each instance acquired the lock, preventing premature release. For Justin Tobler’s alternative proposal to avoid changing the `stack_uptodate()` signature, Nayak argued that lock-checking logic belongs in the function to enforce consistent application and reduce caller error risk. The series remains under review, with these implementation details now the primary focus.

### `git organize` introduced for source tree restructuring
Michael Montalbo introduced `git organize`, a new command designed to restructure Git’s source tree by moving files into subsystem-specific directories (e.g., `odb/`, `refs/`, `pack/`) based on recorded placement rules in a `.gitorganize` file. The 14-patch RFC series includes both the tool itself and a demonstration of its use to reorganize Git’s own codebase, carving out eight subsystems (`odb`, `refs`, `pack`, `diff`, `revision`, `index`, `setup`, `transport`) in separate commits.

The tool externalizes file placement decisions into a tracked configuration file, enabling auditable, reversible reorganizations. It provides two main workflows: a **labeler** assigns semantic labels (e.g., `component=odb`) using project-specific heuristics, and an **organizer** updates references (e.g., `#include` paths, build system entries) after moves. The series demonstrates the tool’s practical application by mechanically moving each subsystem to its own directory and updating all references, leaving the tree buildable after each move.

The patches are well-structured and self-contained, with clear test coverage and documentation. The tool is designed to be opt-in, allowing projects to adopt it incrementally. Reviewers may focus on the labeler’s heuristics (e.g., reliance on commit prefixes), the organizer’s completeness (e.g., handling documentation cross-references), and the tool’s generality for other projects. The series also raises questions about alignment with Patrick Steinhardt’s `libgit.a` series (e.g., nesting subsystems under `lib/`).

### AI-generated content policy debate continues
The policy debate around AI-generated contributions saw new developments, with Junio C Hamano questioning the strictness of Git’s current policy and highlighting precedents for AI-assisted work. The discussion was sparked by Alexey Samsonov’s `utimensat()` migration series, which was generated by an AI tool and manually verified, and Johannes Sixt’s patch to discourage AI-generated contributions in `gitk`.

Weijie Yuan twice surfaced a precedent: Johannes Schindelin has landed commits in `master` with an `Assisted-by` trailer, suggesting prior acceptance of AI-assisted contributions. Junio’s latest emails frame the issue as one of **contributor trust**, implying that the project might tolerate AI-assisted contributions from trusted contributors who disclose their use of tools. This shifts the discussion toward a middle ground that balances policy enforcement with practical trust, though no explicit path forward has been proposed.

The thread remains blocked pending clarification from the Project Leadership Committee (PLC) or a broader community discussion about amending `SubmittingPatches` to address AI-generated content. The distinction between "generated" and "assisted" work, and the sufficiency of `Assisted-by` trailers for disclosure, are likely to recur in follow-up discussions.

## In brief
- **Refname validation tests complete**: Nikolaus Schuetz confirmed that the test suite now exhaustively covers all documented forbidden characters in refnames, both position-independent (e.g., space, `~`, `^`) and position-specific (e.g., dots at start/end of components).
- **Per-worktree stash RFC proposed**: Vladimir Sitnikov proposed a new `stash.ref` configuration knob to make Git’s stash stack per-worktree, defaulting to `refs/stash` but configurable to per-worktree refs (e.g., `refs/worktree/stash`). Kristoffer Haugsbakk questioned whether the problem justifies a new config knob, suggesting automated agents could avoid the stash instead.
- **`gitdatamodel(7)` discoverability improved**: Kristoffer Haugsbakk’s four-patch documentation series adding cross-references to `gitdatamodel(7)` from `git(1)` and `gitglossary(7)` was updated to v2, addressing minor commit message nits. The series is ready for integration.