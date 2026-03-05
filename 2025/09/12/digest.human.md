Here's the Git mailing list daily digest for September 12, 2025:

## The day in brief

A busy Friday with 72 emails across 20 threads saw significant progress on several fronts: the `sparse-checkout clean` feature was approved for merging, signature handling in `fast-import` reached consensus, and the SHA-256 transition planning gained momentum. Meanwhile, a heated discussion about removing `git-new-workdir` revealed deep workflow dependencies, and multiple bug reports surfaced edge cases in Git's merge and patch application logic.

## Notable threads

### `git-new-workdir` removal debate

The planned removal of the `contrib/git-new-workdir` script sparked a substantive discussion about workflow impacts. Gabriel Scherer demonstrated a real-world use case involving 73 remotes that relies on the script's behavior of sharing remote configurations across worktrees - functionality not matched by `git worktree`. While Phillip Wood and Junio Hamano pointed to `--ignore-other-worktrees` as a safer alternative for branch switching, the remote configuration sharing limitation remains unresolved. Junio ultimately defended the removal as necessary technical debt reduction, noting the script's maintenance burden during the reftable implementation. This thread highlights a growing tension between Git's internal cleanup efforts and real-world workflow dependencies.

### Sparse-checkout clean approved

Victoria Dye and Derrick Stolee's `sparse-checkout clean` series received maintainer approval after addressing all feedback. The feature removes tracked-but-sparse directories outside the sparse-checkout scope while preserving untracked files, with safety mechanisms matching `git clean` (force/dry-run requirements). Junio specifically praised the improved documentation and progress on removing `the_repository` usage in the implementation. This marks a significant quality-of-life improvement for sparse-checkout users, particularly in monorepo environments where leftover files can cause performance issues.

### Signature handling for fast-import

Christian Couder's series adding `--signed-commits` to `git fast-import` reached consensus in its second version, with all technical feedback addressed. The implementation shares parsing logic with `fast-export` via new gpg-interface helpers, supporting modes like "verbatim", "strip", and "abort". While Junio questioned starting with commits rather than the more common signed tags, he accepted the phased approach. The thorough test coverage (including SHA-1/SHA-256 cases) and careful error handling position this well for merging, with tag support planned as follow-up work.

### SHA-256 transition planning

Emily Shaffer initiated coordination for the SHA-256 transition, prompting a detailed status update from Brian Carlson. With 93 patches already in their `sha256-interop` branch, Carlson outlined remaining work areas needing assistance: test coverage, `git gc` integration, submodule handling, and repository conversion tools. The email revealed the scale of work remaining (200-300 expected patches) as Git targets SHA-256 as the default for new repositories in 3.0. This thread represents a critical coordination point for one of Git's most significant architectural changes.

### Documentation improvements

Julia Evans' documentation series for `git-push` was merged with post-merge refinements, particularly around upstream branch explanations. The changes consolidate scattered information about upstream tracking into a new "UPSTREAM BRANCHES" section shared across push/pull/fetch docs. Meanwhile, Kristoffer Haugsbakk and Junio sparked a follow-up discussion about better showcasing pathspec capabilities, potentially via a dedicated `gitpathspecs(7)` man page. These changes continue Git's ongoing effort to make its documentation more approachable without sacrificing technical accuracy.

## In brief

**Bundle-URI edge cases** -- Justin Tobler and Toon Claes proposed fixes for git-clone's handling of malformed bundle-URI responses, sparking discussion about whether to warn or silently continue for client misconfigurations.

**Fast-import signature parsing** -- Christian Couder refined error handling style in the already-merged first patch of his signature series, adopting a "return error if failure" pattern per review feedback.

**Merge conflict labeling** -- A feature request proposed enhancing conflict markers to explicitly show which branch is "--ours" and "--theirs" during merge/rebase operations.

**MERGE_HEAD pseudoref behavior** -- Johannes Sixt reported a discrepancy between Git's advice (suggesting `update-ref -d MERGE_HEAD`) and actual behavior (rejecting pseudoref updates since May 2024.

**user.email validation debate** -- A philosophical discussion concluded that Git's lack of email format validation is intentional, with maintainers citing real-world uses like GitHub SSO token storage in the field.

**Merge edge cases** -- Three separate reports from Guo Tingsheng highlighted non-deterministic merge behaviors around indentation preservation, independent method additions, and lost formatting changes - all awaiting reproducible test cases.

**Patch application ambiguity** -- Junio explained `git apply`'s fundamental limitation when patches match multiple locations, suggesting wider context (`-U8`) helps but noting no foolproof solution exists.

## On the radar

**Rust infrastructure** -- The build system patches are in final polishing phase, with only minor cleanup of `Cargo.lock` handling remaining before potential merging.

**ODB transactions** -- Karthik Nayak's reference transaction fixes are addressing edge cases in batched updates, particularly around case-insensitive filesystems.

**git-history RFC** -- D. Ben Knoble identified UI expectations mismatches in the `split` subcommand's diff display behavior during interactive use.