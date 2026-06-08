# Git Mailing List Digest — 2026/06/07 (Sunday)

**The day in brief.** A moderately busy Sunday with 19 emails across 6 threads, dominated by performance optimizations and test infrastructure discussions. The standout developments include a major speedup for `git ls-files` path filtering (60x improvement), final refinements to the priority queue optimization series, and resolution of test implementation concerns for the `diff.<driver>.process` feature. The day also saw an RFC for behavior changes to the experimental `git history reword` command.

## Notable threads

### `diff.<driver>.process` test implementation transition

The long-running `diff.<driver>.process` series, which would allow external tools to inject diff hunks into Git's diff/blame pipelines, reached a key milestone today. Johannes Schindelin identified several issues with the Python-based test implementation, including Windows CI failures and Python version incompatibilities. Michael Montalbo promptly agreed to transition to a C-based test helper, aligning with the project's direction of reducing scripting language dependencies. This addresses the final significant implementation concern for the feature, leaving only verification of Windows fixes and minor refinements before potential inclusion. The core feature design remains uncontested, withstanding months of rigorous review.

### Priority queue optimizations finalized

Kristofer Karlsson's performance optimization series for Git's priority queue implementation saw its final refinements today. René Scharfe provided benchmark results showing a remaining 1% performance gap, leading to Kristofer inlining the flush logic directly into get()/peek() functions. This v3 iteration maintains the 1.7-2.7% speedups on traversal-heavy operations while addressing the last edge case. The series has now incorporated all feedback from three rounds of review, with demonstrated benefits to merge-base calculations and revision walking. The changes affect multiple subsystems but maintain compatibility through careful conversion of all callers to use new access patterns.

### `git ls-files` pathspec filtering optimization

Tamir Duberstein contributed a dramatic optimization for `git ls-files` that filters pathspecs before expensive lstat operations. On a repository with 859k index entries, the `--deleted` check with a pathspec drops from 60.7 seconds to 1.06 seconds — a 50x improvement. The change maintains existing behavior while avoiding unnecessary filesystem checks for non-matching paths. The patch includes thorough performance and functional tests, making it particularly impactful for large repositories with top-level pathspecs or leading wildcards. A parallel discussion about LLM attribution in commit messages was cleanly resolved by referencing Git's existing AI contribution guidelines.

## In brief

**`git describe` tag-only optimization** — A patch limits ref iteration to tags by default in `git describe`, showing a 3.1x speedup (196ms -> 63ms) in repositories with many non-tag refs. The change avoids iterating over irrelevant refs when `--all` isn't specified.

**`git history reword` behavior changes** — Pablo Sabater proposes two changes to the experimental command: aborting when commit messages match (unlike amend/rebase) and adding explicit success feedback. The RFC seeks input on whether diverging from established behavior is justified by the command's different constraints.

**Priority queue benchmark analysis** — René Scharfe provided additional benchmarking showing that while cascade optimization works best for general extraction, sift-down remains superior for root operations. Kristofer Karlsson confirmed this hybrid approach will remain relevant even after planned API changes.