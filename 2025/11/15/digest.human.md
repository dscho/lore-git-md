Here's the daily digest for November 15, 2025:

## The day in brief
A moderately busy day with 45 emails across 12 threads, featuring significant progress on several technical fronts. The most notable developments include final approvals for the submodule hash validation series and continued refinement of UTF-8 display alignment, while the xdiff refactoring discussion reached consensus on type documentation.

## Notable threads

### Submodule hash validation approved
brian m. carlson's v2 series preventing mixed-hash-algorithm submodules received final approval from both Junio Hamano and Jeff King. The patches add explicit validation in `index_path()` to block submodule additions when parent and child use different hash algorithms (SHA-1 vs SHA-256), with comprehensive test coverage. This resolves a long-standing concern about repository corruption possibilities, though limitations remain for shallow clones and nested submodules.

### UTF-8 alignment refinements
Jiang Xin's v2 series fixing CJK character alignment in repository structure output underwent detailed review, with maintainers suggesting alternative implementation approaches. Phillip Wood questioned encoding assumptions while Junio Hamano emphasized the need for end-to-end testing beyond unit tests. The discussion revealed this as part of Git's broader UTF-8 handling challenges, with the series now considering both `strbuf_utf8_align()` and manual `printf()`-based solutions.

### xdiff type documentation consensus
The xdiff refactoring thread reached agreement on how to document C/Rust type mappings after extensive discussion of `char` signedness semantics. Ramsay Jones, Ben Knoble, and Junio Hamano aligned on properly characterizing C's implementation-defined behavior while accepting that `-Wsign-compare` warnings reflect compiler limitations rather than true type safety issues. This clears a key obstacle for the series' progress.

### Hook subsystem memory leaks
Junio Hamano identified two memory leaks in Adrian Ratiu's receive-pack hook conversion - one in `feed_receive_hook_cb()` allocations and another in `state->buf` handling during EPIPE errors. The feedback comes as this mature refactoring nears completion, with attention shifting from API design to implementation robustness.

## In brief

**Bash prompt quiet mode** -- Kiril Ivanov introduced a series adding quiet upstream indicators to `__git_ps1`, suppressing the '=' sign when branches are synchronized while maintaining divergence warnings.

**Lisp userdiff driver** -- Scott L. Burson proposed a new "lisp" driver for Common Lisp, prompting discussion about whether to merge its patterns with the existing Scheme driver.

**gitk tracking ref fix confirmed** -- Johannes Sixt verified that gitk's handling of stale tracking references was fixed in v2.52.0, resolving a regression reported against v2.51.0.

**Fetch tag cleanup flow** -- Karthik Nayak refined error handling for non-atomic fetches, ensuring proper cleanup while fixing tag commitment issues.

**Commit --committer v4** -- ZheNing Hu's controversial `--committer` flag proposal reached v4 with improved implementation but persistent maintainer skepticism about its necessity.

**ASan timestamp parsing** -- Jeff King fixed ASan warnings in timestamp parsing by avoiding `strtoumax()` on non-NUL-terminated buffers, while pushing back on generalized parsing utilities.

## On the radar

**git history feedback** -- Early user reports on the experimental `git history` command are positive for basic "reword" operations, though architectural placement questions remain unresolved.

**What's cooking** -- Junio's latest status report flagged several topics needing attention, though the full digest wasn't included in today's traffic.